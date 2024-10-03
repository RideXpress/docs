# CI/CD Pipeline Documentation

---

## 1. Pull from Design Center

### Introduction
This phase involves retrieving the latest projects from the Design Center to ensure local development is up to date.

### Process
1. **List all Design Center Projects:** Identify all available projects.
2. **For each project:**
   - **Does the project exist locally?**
     - **Yes:** Download content into the `{project}/src/main/resources/api` directory.
     - **No:** Download the project inside the design-center directory.
3. **Open Pull Request to review latest changes.**

### Diagram Explanation
This diagram shows how projects are pulled from the Design Center, checking if they exist locally, and opening a pull request to review the updates.

[View Pull from Design Center Diagram](./pull-from-design-center.svg)

---

## 2. Push to Design Center

### Introduction
After making changes to the project, the next step is to push these changes back to the Design Center.

### Process
1. **For each project inside the Design-Center directory:**
   - **Publish project to Exchange.**

### Diagram Explanation
The diagram illustrates how projects from the Design Center are published to Anypoint Exchange after local changes.

[View Push to Design Center Diagram](./push-to-design-center.svg)

---

## 3. Build and Deploy

### Introduction
This phase compiles and deploys the application to various environments after the code has been pulled and updated.

### Process
1. **For each change in latest Commit:** The pipeline triggers automatically.
2. **Run MUnits:** Executes unit tests to validate the code.
3. **Publish application to Exchange.**
4. **Publish application to API Manager.**
5. **Create API SLAs.**
6. **Configure API Policies.**
7. **Pull API configuration from API Manager.**
8. **Deploy application to Runtime Manager.**
9. **Publish to GitHub Packages.**

### Diagram Explanation
This diagram shows the full process of building and deploying the application, including unit testing, publishing to Anypoint Exchange, API Manager, and Runtime Manager.

[View Build and Deploy Diagram](./build-and-deploy.svg)

---

## 4. Release

### Introduction
This phase finalizes the release process, promoting stable artifacts and making the API available for consumption by users. It also tags the code for version control.

### Process
1. **Read artifacts to release from configuration file:** The pipeline reads the artifacts and versions to be promoted based on the configuration.
2. **For each artifact:**
   - **Update Asset in Exchange to mark it as Stable.**
   - **Promote API in API Manager:** Move the API to production or a stable environment.
   - **Promote Application in Runtime Manager:** Promote the application deployment to the designated runtime environment.
3. **Validate deployment:** Ensure that the deployment was successful and the application is running correctly.
4. **Tag code:** Tag the repository in GitHub with the new release version.

### Diagram Explanation
This diagram outlines the release process, from reading the configuration file to promoting the APIs and tagging the code.

[View Release Diagram](./release.svg)

---
