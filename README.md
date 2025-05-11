# C.U.M - Commit User Modification

It's not just a tool, it's an experience. C.U.M. diligently observes your coding endeavors and eloquently summarizes them, ensuring every brilliant modification you commit is duly noted and chronicled.

<div align="center">
  <img src="CUM.gif" alt="CUM">
</div>

## What is C.U.M.?

**C.U.M. (Commit User Modification)** is an automated system that enhances your Git workflow by:

1.  **Analyzing your code changes** (the "User Modifications") upon every `git commit`.
2.  **Generating a high-level summary** of these changes using a powerful Large Language Model (LLM).
3.  **Appending this summary** to a dedicated LaTeX report file (`CUM_report/commit_log.tex`) within your repository.
4.  **Automatically amending the commit** to include this updated report, ensuring the summary is perfectly in sync with the changes it describes.

Essentially, C.U.M. makes your commits self-documenting in a beautifully formatted LaTeX report, all happening seamlessly in the background!

## Core Features

* **Automatic Commit Summarization:** Leverages AI to understand and describe your code changes.
* **LaTeX Report Generation:** Creates and maintains a `commit_log.tex` file, chronicling your project's evolution with AI-generated insights.
* **Seamless Git Integration:** Uses a `post-commit` Git hook for a smooth, behind-the-scenes operation.
* **In-Sync Documentation:** By amending the commit, the summary for commit `X` is included *within* commit `X` itself. No more out-of-sync generated files!
* **Configurable LLM Backend:** Designed to interface with LLM inference engines like SERAPHIM.

## How It Works

C.U.M. is deployed via an installation script (`install_cum.sh`) that sets up a `post-commit` hook in your local Git repository. Here's the flow:

1.  You make your changes and run `git commit`.
2.  After your initial commit is successfully created, the `post-commit` hook automatically triggers.
3.  The hook script:
    * Extracts the diff of the commit you just made.
    * Sends this diff to your configured LLM API endpoint (e.g., hosted by SERAPHIM) to get a summary.
    * Formats the summary and appends it as a new section to the `CUM_report/commit_log.tex` file.
    * Adds the updated `commit_log.tex` to the staging area.
    * Amends the commit you just made (`git commit --amend --no-edit -C HEAD`), incorporating the updated LaTeX report directly into it.
4.  The result? Your commit now includes its own AI-generated summary in the report, and your working directory remains clean. You can then `git push` this "complete" commit.

## Prerequisites

* **Linux-based System:** The installation script primarily uses `apt` for package management, with attempts for `yum` and `brew`.
* **Git:** Obviously!
* **jq & curl:** These command-line tools are used for interacting with the API and handling JSON. The installation script will attempt to install them if they are missing.
* **Running SERAPHIM Inference Engine:** C.U.M. sends its summarization requests to an LLM served via the [**SERAPHIM engine**](https://github.com/Anderson-L-Luiz/SERAPHIM). SERAPHIM, which utilizes vLLM for high-performance inference, must be running and accessible, serving the model specified in the C.U.M. configuration (default: `deepseek-ai/deepseek-moe-16b-chat`).
* **LaTeX Distribution:** To compile the generated `CUM_report/commit_log.tex` file into a viewable document (e.g., PDF using `pdflatex`). Make sure you have common packages like `lmodern`, `courier`, `hyperref`, `parskip`, `amsmath`.

## Installation

1.  **Get the Script:** Clone this repository or download the `install_cum.sh` script into the root directory of your target Git project.
2.  **Navigate to Your Project:** Open your terminal and `cd` into the root directory of your Git project where you want to install C.U.M.
3.  **Run the Installer:**
    ```bash
    bash install_cum.sh
    ```
    The script will:
    * Check for Git and the project root.
    * Attempt to install `jq` and `curl` if not found (may require `sudo` password).
    * Create the `CUM_report` directory and the initial `commit_log.tex` file if they don't exist.
    * Install the `post-commit` hook script into your repository's `.git/hooks/` directory and make it executable.
    * Remove any old C.U.M. `pre-push` hook if detected.

## Configuration

The C.U.M. system relies on a few settings primarily found within the hook script itself:

* **`VLLM_API_URL`**: The full URL to your SERAPHIM/vLLM API endpoint.
    * Default in script: `http://10.16.246.2:8001/v1/chat/completions`
* **`MODEL_NAME`**: The name of the model as recognized by your SERAPHIM/vLLM instance.
    * Default in script: `deepseek-ai/deepseek-moe-16b-chat`
* **`MAX_TOKENS`**: Maximum tokens for the LLM summary.
    * Default in script: `700`

**To change these defaults:**

1.  **Before installation:** Modify these variables directly in the `install_cum.sh` script within the `cat << 'HOOK_SCRIPT_EOF' > "$HOOK_PATH"` block.
2.  **After installation:** Edit the installed hook file directly at `.git/hooks/post-commit` in your repository.

## Usage

Once C.U.M. is installed, your Git workflow for committing changes remains largely the same:

1.  Make your code modifications.
2.  Stage your changes: `git add <your-files>`
3.  Commit your changes: `git commit -m "Your awesome commit message"` (or any way you normally commit).

    * Upon successful commit, the C.U.M. `post-commit` hook will run automatically. You'll see output in your terminal indicating it's analyzing the commit, contacting the LLM, and amending the commit.
    * The `CUM_report/commit_log.tex` file will be updated with the new summary and this change will be included in the commit you just made.

4.  **Push your changes:** `git push`
5.  **View the Report:**
    * Navigate to the `CUM_report` directory.
    * Compile the LaTeX file using your preferred LaTeX distribution, for example:
        ```bash
        pdflatex commit_log.tex
        pdflatex commit_log.tex # Run twice for table of contents and references
        ```
    * Open the resulting `commit_log.pdf`.

## Troubleshooting

* **Hook not running:**
    * Ensure `install_cum.sh` completed successfully.
    * Verify that `.git/hooks/post-commit` exists and is executable.
    * Check `git config core.hooksPath`. If set, Git looks for hooks there instead of `.git/hooks/`. Unset it for your repo (`git config --unset core.hooksPath`) or install the hook to the custom path.
* **API Errors (e.g., 404, connection refused, parsing issues):**
    * Verify your SERAPHIM/vLLM server is running and accessible at the `VLLM_API_URL` configured in the hook.
    * Ensure the API endpoint (e.g., `/v1/chat/completions`) is correct for your server setup. You can often find available endpoints by visiting `http://<your-server-ip>:<port>/docs` (if provided by SERAPHIM/vLLM).
    * Check that `MODEL_NAME` matches how the model is identified by the server.
    * Review the SERAPHIM/vLLM server logs for any errors when the hook attempts a request.
    * The hook will output `curl` error codes and snippets of the API response if it fails to parse the summary, which can help diagnose issues.
* **LaTeX Compilation Errors:**
    * Ensure you have a working LaTeX distribution with standard packages installed (e.g., `lmodern`, `courier`, `hyperref`, `parskip`, `amsmath`).
    * Check `commit_log.log` for specific LaTeX errors after attempting compilation.

## How to clean out your C.U.M.

If you wish to remove C.U.M. from your repository and stop the automatic summarization:

1.  **Delete the post-commit hook:**
    ```bash
    rm .git/hooks/post-commit
    ```
    (If you are unsure of the exact path, it's the `HOOK_PATH` referred to in the installation script, typically `your_project_root/.git/hooks/post-commit`)

2.  **(Optional) Delete the generated report directory:**
    If you no longer want the `CUM_report` directory and its contents (`commit_log.tex`, compiled PDFs, etc.), you can remove it:
    ```bash
    rm -rf CUM_report
    ```
    Remember to `git add .` and `git commit` this deletion if you want to remove it from the repository's history going forward.

3.  **(Optional) Advanced: Removing previously generated LaTeX content from Git history:**
    If you want to entirely remove all traces of the `CUM_report` directory and `commit_log.tex` from your project's *past* Git history, this requires more advanced Git operations like `git filter-repo` (recommended over `git filter-branch`) or the BFG Repo-Cleaner. **These operations rewrite history and should be done with extreme caution, especially if the repository is shared.** Make sure to back up your repository before attempting such operations.

---

Embrace the C.U.M. workflow and let your commits speak for themselves!
