# Gavan Lamb Website

## Documentation
### Produce artifacts 
1. Install dependencies
   ```powershell
   pip install mkdocs
   pip install mkdocs-material
   pip install mkdocs-mermaid2-plugin
   pip install mkdocs-table-reader-plugin
   ```

2. Build
   ```powershell
   cd docs
   mkdocs build
   ```

3. Serve
   ```powershell
   cd docs
   mkdocs serve
   ```

## Versioning
### Generate version
1. Install dependencies
   ```powershell
   dotnet tool update -g nbgv
   ```

2. Generate version
   ```powershell
   ngbv cloud --all-vars
   ```
