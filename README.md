# InflorAI

## Project Description
InflorAI is a web tool that automates the conversion of non-uniform text offers from flower suppliers into a consistent, easy-to-read price list. The application allows users to:
- Paste raw text data from supplier offers.
- Set global pricing parameters, including currency conversion rates, net margins, and transport costs.
- Automatically extract and recalculate item prices based on predefined rules.
- Manually verify and edit the generated price list in an editable text field.
- Generate a final price list in a tab-separated format.
- Maintain a query history for authenticated users.

This solution aims to reduce manual errors, increase efficiency, and provide a seamless experience in managing supplier offers.

## Tech Stack
- **Frontend:** Astro 5, React 19, TypeScript 5, Tailwind CSS 4, Shadcn/ui
- **Backend:** Supabase (PostgreSQL, authentication, and database services), Openrouter.ai for AI-enabled features
- **CI/CD & Hosting:** GitHub Actions for continuous integration and DigitalOcean (via Docker) for deployment

## Getting Started Locally
### Prerequisites
- **Node.js:** Ensure you have Node.js version specified in `.nvmrc` (v22.14.0)
- **Package Manager:** npm (or yarn, if preferred)

### Installation and Setup
1. Clone the repository:
   ```bash
   git clone https://github.com/jankoEM/10x-project
   cd your-repo
   ```
2. Install dependencies:
   ```bash
   npm install
   ```
3. Start the development server:
   ```bash
   npm run dev
   ```

## Available Scripts
In the project directory, you can run:
- **`npm run dev`**: Runs the app in development mode.
- **`npm run build`**: Builds the app for production.
- **`npm run preview`**: Serves the production build locally.
- **`npm run astro`**: Runs Astro CLI commands.
- **`npm run lint`**: Lints the project files.
- **`npm run lint:fix`**: Lints and automatically fixes issues.
- **`npm run format`**: Formats the codebase using Prettier.

## Project Scope
The MVP of InflorAI includes:
- A user interface for pasting raw offer text and setting initial parameters.
- Automatic extraction and pricing calculation based on rules (including VAT application, currency conversion, and cost adjustments for product dimensions).
- Editable outputs in a text field allowing manual correction.
- Basic user authentication and secure query history management.
- Limitations: Advanced data validation, extensive error handling beyond the MVP scope, and integrations with external systems like ERP/CRM are not included.

## Project Status
This project is in the MVP phase under active development. 
## License
This project is licensed under the MIT License.