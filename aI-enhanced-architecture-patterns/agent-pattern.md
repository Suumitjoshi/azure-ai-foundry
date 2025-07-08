The Agent Pattern in AI refers to building autonomous or semi-autonomous AI systems that:
- Understand goals (through user inputs or triggers)
- Plan next steps (with reasoning via LLMs like GPT-4)
- Act by calling tools (functions, APIs, workflows)
- Remember via memory (to maintain context)
- Iterate until task is complete

These agents can make decisions dynamically and interact with external systems, enabling intelligent automation.

## .NET Code

#  Program.cs
var builder = Kernel.CreateBuilder()
    .AddAzureOpenAIChatCompletion("gpt-4", "endpoint", "key")
    .AddAzureMemoryStore(new VolatileMemoryStore())
    .AddFromType<ComplianceDocsPlugin>();

var kernel = builder.Build();

var planner = new FunctionCallingStepwisePlanner();
var plan = await planner.CreatePlanAsync(kernel, "Answer ISO27001 questions from docs.");

var result = await plan.InvokeAsync(kernel);
Console.WriteLine(result.GetValue<string>());


# PlannerAgent.cs

public class PlannerAgent
{
    private readonly Kernel _kernel;
    private readonly IStepwisePlanner _planner;

    public PlannerAgent(Kernel kernel)
    {
        _kernel = kernel;
        _planner = new FunctionCallingStepwisePlanner();
    }

    public async Task<string> RunAsync(string goal)
    {
        var plan = await _planner.CreatePlanAsync(_kernel, goal);
        var result = await plan.InvokeAsync(_kernel);
        return result.GetValue<string>();
    }
}


# ComplianceDocsPlugin.cs 
public class ComplianceDocsPlugin
{
    [KernelFunction] 
    // It is an annotation used in Semantic Kernel , Once a method is decorated with [KernelFunction], it becomes part of the toolset the agent can reason about and invoke during execution.

    public async Task<string> SearchComplianceDocs(string question)
    {
        // Use Azure AI Search or call external API
        return "Fetched relevant paragraph from ISO 27001 docs...";
    }
}


## Why is [KernelFunction] Important?
| Purpose                            | Description                                                                                                  |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| 🔧 **Expose tool functions**       | Lets your agent access C# methods like `SearchComplianceDocs`, `GetWeather`, `QueryDatabase`, etc.           |
| 🧠 **Enable reasoning & planning** | The LLM sees the method signature and can choose to call it when needed (using planners or function calling) |
| 📚 **Provide metadata**            | You can add descriptions and parameter help (to guide LLMs)                                                  |
| 🔄 **Auto-register plugins**       | These methods get picked up automatically during plugin registration (`AddFromType<T>()`)                    |


# Code
public class ComplianceDocsPlugin
{
    [KernelFunction(Name = "SearchComplianceDocs", Description = "Search ISO/GDPR docs")]
    public async Task<string> SearchComplianceDocs(string question)
    {
        // Actual search logic using Azure AI Search
        return $"Found info for: {question}";
    }
}

# Registration
builder.AddFromType<ComplianceDocsPlugin>();

# Once registered, the LLM will be able to see and call SearchComplianceDocs() when needed during the reasoning phase.

## Real-World Applications

🧾 A. Compliance & Audit Assistant
What: Agent reads ISO, GDPR, NIST documents
How: Uses Azure AI Search to fetch paragraphs, GPT-4 to summarize
Benefit: Faster compliance assessments for auditors

🩺 B. Medical Diagnosis Assistant
What: Doctor inputs symptoms, agent asks follow-up, calls APIs
How: GPT + medical DB APIs + search
Benefit: AI-assisted triage

🏗️ C. Smart DevOps Agent
What: Ask "Why is service X down?" → Agent queries logs, status, alerts
How: GPT + Prometheus + Datadog API
Benefit: Automate RCA (Root Cause Analysis)

🧑‍💻 D. AI Code Reviewer Agent
What: Pulls PR code → checks against best practices
How: GPT + StyleCop + Repo APIs
Benefit: LLM-powered code analysis

📞 E. Customer Support Agent
What: Handles support tickets using past knowledge base and actions
How: GPT + ticket system + CRM APIs
Benefit: Reduce human workload


