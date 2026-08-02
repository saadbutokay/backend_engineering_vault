```bash
poetry --version                 # Verify installation
poetry new project_name          # Create new project
poetry init                      # Initialize in existing folder
poetry config virtualenvs.in-project true # adds venv in folder
poetry install                   # Install all dependencies
poetry add package_name          # Add a dependency
poetry add --group dev pytest    # Add dev dependency
poetry remove package_name       # Remove a dependency
poetry update                    # Update all packages
source <(poetry env activate)    # Activate Poetry's venv
poetry show                      # List installed packages
poetry show --tree               # Show dependency tree
poetry run python script.py      # Run inside venv
poetry build                     # Build distributable package
poetry publish                   # Publish to PyPI
poetry env info                  # Show venv info
poetry lock                      # Regenerate lock file
exit                             # Leave the venv shell
```

