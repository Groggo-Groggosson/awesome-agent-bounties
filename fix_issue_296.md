# Fix for #296

To solve this issue, we need to create a bash script that automatically generates a structured CHANGELOG from git commit messages. Here's a complete, working solution:

1. **Create a `generate_changelog.sh` script**:
   This script will read the latest commits and generate a formatted CHANGELOG.

2. **Add the script to the repository**:
   Place the script in an appropriate directory, such as a `scripts` folder.

3. **Make the script executable**:
   Ensure the script has execute permissions.

4. **Test the script**:
   Run the script locally to ensure it generates the expected output.

Here's the complete code: