# 🧠 Copilot Instruction Files Library

This repository contains a collection of **architecture-specific GitHub Copilot instruction files** that enforce clean project structures, coding standards, and best practices across different kinds of software projects.

These instruction files help GitHub Copilot generate **consistent, scalable, production-grade code** for any architecture you choose.

---

# 📁 Repository Structure

Each architecture folder includes a `.github/copilot-instructions.md` file that can be copied directly into any project.

Current structure:

copilot-instruction-files/
class-library architecture/
.github/
copilot-instructions.md


More architectures will be added soon.

---

# 🚀 How to Use

### 1️⃣ Choose an architecture  
Go to any folder, for example:

class-library architecture/.github/copilot-instructions.md


### 2️⃣ Copy the `copilot-instructions.md` file  
Paste it into **your project’s**:

.your-project/
.github/
copilot-instructions.md


### 3️⃣ Restart VS Code / Cursor  
GitHub Copilot will automatically start using those rules for:

- Code generation  
- Folder scaffolding  
- API design  
- Service & repository patterns  
- DTO and mapping conventions  
- Testing  
- Logging  
- Clean coding practices  

### 4️⃣ Start building your project  
Copilot will now follow **the exact architecture guidelines** from the selected instructions file.

---

# 🏗️ Available Architectures

### ✔ Class Library Architecture (C# / .NET)
- Domain project  
- Application project  
- Infrastructure project  
- API project  
- Utilities/shared project  
- EF Core + PostgreSQL  
- Repository + Service layers  
- Clean Architecture principles  

(Other architectures will be added soon.)

---

# 🤝 Contributing

You can add more architecture patterns by:

1. Creating a new folder inside `copilot-instruction-files/`
2. Adding a `.github/copilot-instructions.md` file
3. Following the same structure and formatting
4. Creating a pull request

---

# ⭐ Support

If you find this repo helpful:
- ⭐ Star the repository  
- Share it with other developers  
- Suggest new architectures in Issues  

---

Made with ❤️ to help developers build **high-quality, AI-assisted software architectures**.
