# unused-task-warning
When using dependency injection and async-await pattern it is possible to end up with an interface with a method that returns a Task. If this interface method is used in a synchronous method there is a likelihood that it will erroneously be run as a fire and forget method (which will not trigger inbuilt warning CS4014). In this situation this analyser generates a warning.

Can both be used as a Visual Studio extension or preferably as a project analyser available from NuGet.

Example:

	using System.Threading.Tasks;

	namespace AsyncAwaitProblem
	{
		public interface ICallee
		{
			bool ProblemSolved { get; }
			Task SolveProblemAsync();
		}

		public class Callee : ICallee
		{
			public bool ProblemSolved { get; set; }

			public async Task SolveProblemAsync()
			{
				await Task.Delay(10);
				ProblemSolved = true;
			}
		}
		
		public class Caller
		{
			public bool DoCall()
			{
				ICallee xxx = new Callee();
	
				// This analyser will give a warning at the following line
				xxx.SolveProblemAsync(); // This is most likely an undesired fire and forget. 
	
				return xxx.ProblemSolved; // Will return false - we expected it to return true
			}
		}
	}


Note that this analyser currently only checks for current known awaitable types (ex. types `Task`, `ValueTask` and `ConfiguredTaskAwaitable` (the type returned when using the `ConfigureAwait` method)). If another ['Awaitable'](https://blogs.msdn.microsoft.com/pfxteam/2011/01/13/await-anything/) type is returned this analyser will not give the warning. This might be fixed in a future version.

Available from NuGet here: https://www.nuget.org/packages/Lindhart.Analyser.MissingAwaitWarning/

Available from Visual Studio marketplace here: https://marketplace.visualstudio.com/items?itemName=Lindhart.missingAwaitWarning#overview

For information on how to use analyzers check here: https://docs.microsoft.com/en-us/visualstudio/code-quality/roslyn-analyzers-overview?view=vs-2019
