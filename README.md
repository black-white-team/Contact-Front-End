# 📋 Contact Management Project

## 💡 Overview


This project is a web-based contact management application built using Vue.js and Element Plus. It allows users to manage their contacts, including phone numbers, email addresses, social media handles, and physical addresses. Users can perform CRUD (Create, Read, Update, Delete) operations on contact information and use features such as import/export, bookmarking, and search.

The application is styled to resemble an Apple-like interface, providing an intuitive and user-friendly experience.

## 💪 Features

- **Add, Edit, and Delete Contacts**: Manage multiple fields like phone numbers, email addresses, social media handles, and physical addresses.
- **Search Contacts**: Search for contacts based on specific criteria.
- **Bookmark Contacts**: Mark contacts as favorites and filter them easily.
- **Import/Export Contacts**: Import contacts from an Excel file or export all contacts to Excel.
- **Pagination**: Browse through the list of contacts with pagination support.
- **Data Validation**: Ensure that data entered for phone numbers, emails, etc., is valid.

## 💻 Technologies Used

- **Vue.js**: A progressive JavaScript framework used for building the user interface.
- **Element Plus**: A Vue 3 UI framework that provides a rich set of UI components, making it easier to build beautiful interfaces.
- **Axios**: A promise-based HTTP client for making requests to the server.
- **SCSS**: Used for styling the application, following an Apple-inspired design aesthetic.

## 🛠️ Project Structure

- **`<template>`**: Defines the UI layout, containing buttons for operations, search bar, data table, and dialogs for contact management.
- **`<script>`**: Contains the core logic for managing state, handling CRUD operations, pagination, and importing/exporting data.
- **`<style scoped>`**: Defines CSS styles for the components, providing a consistent design throughout the application.

## 📝 Setup Instructions

### 🛡️ Prerequisites

Before setting up the project, make sure you have the following installed:

- **Node.js**: Version 12 or above.
- **npm** or **yarn**: For managing dependencies.

### 📥 Installation

1. **Clone the repository**:

   ```bash
   git clone <repository-url>
   cd contact-management
   ```

2. **Install dependencies**:

   ```bash
   npm install
   # or
   yarn install
   ```

3. **Run the development server**:

   ```bash
   npm run serve
   # or
   yarn serve
   ```

4. **Access the application**:
   Open your browser and navigate to `http://localhost:8080` to view the contact management interface.

### 🐛 API Configuration

The application uses Axios to make API requests. Ensure that the backend server is properly set up and accessible at `/api/user`. If the API URL differs, modify the Axios requests accordingly in the `<script>` section.

### 🔨 Building for Production

To create a production build of the application:

```bash
npm run build
# or
yarn build
```

This will create an optimized version of the app in the `dist` directory.

## 🔢 Usage Instructions

- **Adding a New Contact**: Click the **New** button and fill in the form fields (e.g., phone numbers, email addresses). Click **Confirm** to save.
- **Editing an Existing Contact**: Click the **Edit** button next to a contact, modify the details, and save the changes.
- **Deleting a Contact**: Click the **Delete** button next to a contact, and confirm the deletion.
- **Bookmarking a Contact**: Click the star icon to bookmark a contact for easy access.
- **Importing Contacts**: Click the **Import** button and select an Excel file (.xls or .xlsx) to import.
- **Exporting Contacts**: Click the **Export** button to download all contacts in an Excel file.

## 📏 File Structure Overview

- **`Home.vue`**: Main Vue component containing all functionality for managing contacts.
- **API Requests**: The application interacts with the backend using Axios to make GET, POST, PUT, DELETE, and PATCH requests.
- **Dialogs and Forms**: Uses Element Plus dialogs to manage adding and editing of contacts, including dynamic handling for different contact types (phone numbers, emails, social media, etc.).

## 💎 Customization

### 🎨 Styles

The styles are defined in the `<style scoped>` block and include custom buttons, tables, dialogs, and pagination. Feel free to adjust the styles to match your design requirements.

### 👤 Contact Fields

The contact fields include phone numbers, email addresses, social media handles, and physical addresses. You can add more fields or modify existing ones by adjusting both the UI and script sections.
