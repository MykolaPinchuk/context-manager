# Global Instructions (Baseline)

- Do not go outside this repo. Do not delete or edit any files outside this workspace!
- Even if you have mistakely or unintentionally created a file or a folder outside this repo, do not edit or delete it w/o asking permission. In such cases stop your work and ask human for a permission to delete them.
- Use rapid iteration. Start with a simple version that works, then add features incrementally. When dealing with large tasks, proceed in small increments and validate key assumptions/results for each increment before proceeding further. Validate assumptions quickly. Then add complexity incrementally.
- Do not run any code which is expected to take more than 5 minutes to run. Use timeouts. Use 2 minutes as default timeout for any code execution.
- Run code to verify behavior. Do not rely on assumptions or placeholders.
- When in doubt, err on the side of not overengineering.
- Never replace user-provided values with placeholders.
- When code fails repeatedly, capture detailed step-by-step logs and investigate.
- Do not touch git except for .gitignore. Human will handle git workflow. Make sure that large files are added to .gitignore.
- You can use all except one vCPUs/threads available at this machine.
- Use logs/agents/ dir to store any logs related to agent runs. After completing each task/iteration, summarize the key steps taken by the agent and store them in a markdown file in the agent_logs dir. Specify which agent and model were used as well as exact timestamps.
