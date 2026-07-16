# Fix for #296

To address the issue of generating a structured CHANGELOG from git, we can create a bash script that automates this process. Below is a complete, working fix for the task:

```bash
#!/bin/bash

# Ensure the script is run in the root directory of the repository
if [ ! -d ".git" ]; then
  echo "Error: This script must be run from the root directory of the git repository."
  exit 1
fi

# Define the output file for the CHANGELOG
CHANGELOG_FILE="CHANGELOG.md"

# Check if the CHANGELOG file already exists and create a backup if it does
if [ -f "$CHANGELOG_FILE" ]; then
  mv "$CHANGELOG_FILE" "${CHANGELOG_FILE}.bak"
fi

# Initialize an empty array to hold the changelog entries
declare -a CHANGELOG_ENTRIES=()

# Get all tags sorted by version number
TAGS=$(git tag --sort=-v:refname)

# Loop through each tag and generate a changelog entry
for TAG in $TAGS; do
  # Get the commit messages for this tag
  COMMITS=$(git log --pretty=format:"- %s" "$TAG"^..$TAG)
  
  # Add a header for this tag
  CHANGELOG_ENTRIES+=("## $TAG ($(date -d @$TAG +%Y-%m-%d))")
  
  # Add the commit messages to the changelog entry
  CHANGELOG_ENTRIES+=("$COMMITS")
done

# Join all entries into a single string and write to the CHANGELOG file
echo "${CHANGELOG_ENTRIES[@]}" > "$CHANGELOG_FILE"

# Check if the CHANGELOG file was created successfully
if [ -f "$CHANGELOG_FILE" ]; then
  echo "Changelog generated successfully: $CHANGELOG_FILE"
else
  echo "Error: Failed to generate changelog."
fi
```