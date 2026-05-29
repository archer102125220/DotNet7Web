# DotNet7Web

This is a .NET 7 Web (Razor Pages) project primarily intended for learning purposes.

## Environment
- **IDE**: Visual Studio 2022 for Mac
- **Framework**: .NET 7

## How to Run the Project

### Via IDE (Recommended)

If you are using **Visual Studio 2022 for Mac** (or another supported IDE):
1. Double-click the `DotNet7Web.sln` solution file in the project directory to open the project.
2. Click the "Run" button at the top or press `Cmd + Enter` to build and start the project.
3. Your default web browser will automatically open and navigate to the local development server address.

### Via Command Line (CLI)

If you prefer using the Terminal or .NET CLI to run this project, make sure you have the .NET 7 SDK installed.

1. **Navigate to the project root directory**:
   ```bash
   cd /path/to/DotNet7Web
   ```
   *(Please replace the path with the actual location where you stored the project)*

2. **Restore NuGet packages**:
   ```bash
   dotnet restore
   ```

3. **Run the project**:
   ```bash
   dotnet run --project DotNet7Web/DotNet7Web.csproj
   ```

4. **View in browser**:
   Once the project has started, the terminal will output messages similar to the following:
   ```text
   info: Microsoft.Hosting.Lifetime[14]
         Now listening on: https://localhost:xxxx
   info: Microsoft.Hosting.Lifetime[14]
         Now listening on: http://localhost:yyyy
   ```
   Copy the `https://localhost:xxxx` URL from the output and paste it into your browser to view the website.

### Stop the Project
Press `Ctrl + C` in the terminal to stop the running application.
