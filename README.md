# ExamApp

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

## Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Installation](#installation)
  - [Backend Installation](#backend-installation)
  - [Frontend Installation](#frontend-installation)
- [Usage](#usage)
- [API Documentation](#api-documentation)
- [Database Schema](#database-schema)
- [Contribution Guidelines](#contribution-guidelines)
- [License](#license)

## Project Overview

ExamApp is an open-source examination platform designed to create and manage exams with dynamic questions. Questions are provided at the time of exam creation, eliminating the need to specify an exam ID beforehand. The platform supports multiple question types, rich content, and a modular frontend with customizable skins, making it versatile for various educational needs.

## Features

- **Dynamic Questions**: Add questions directly when creating an exam, with automatic association handled by the backend.
- **Multiple Question Types**: Supports Multiple Choice Questions (MCQ), fill-in-the-blank, and text responses.
- **Rich Content**: Incorporates text, images, tables, and math formulas in questions and options.
- **Customizable Skins**: Modular frontend with skins tailored to different exam types (e.g., math, language).
- **Scalable Backend**: Compatible with multiple relational databases for flexibility and growth.
- **RESTful API**: Easy integration with external systems via a well-defined API.

## Tech Stack

- **Backend**: Django, Django REST Framework
- **Frontend**: React, Styled Components
- **Database**: Relational databases (PostgreSQL recommended; compatible with MySQL, SQLite, etc.)

## Installation

### Backend Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/your-username/ExamApp.git
   cd ExamApp/backend
   ```

2. **Set Up Virtual Environment**:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

4. **Apply Database Migrations**:
   ```bash
   python manage.py migrate
   ```

5. **Create a Superuser**:
   ```bash
   python manage.py createsuperuser
   ```

6. **Run the Development Server**:
   ```bash
   python manage.py runserver
   ```

### Frontend Installation

1. **Navigate to Frontend Directory**:
   ```bash
   cd ../frontend
   ```

2. **Install Dependencies**:
   ```bash
   npm install
   ```

3. **Start the Development Server**:
   ```bash
   npm start
   ```

## Usage

1. **Create an Exam with Questions**:
   - Use the frontend interface or API to create an exam, providing its details and questions in one go. The backend automatically links questions to the exam.
   
2. **Take an Exam**:
   - Select an available exam, answer the questions, and submit responses for scoring.

3. **View Results**:
   - Access scores and feedback (if implemented) via the frontend or API.

See the [API Documentation](#api-documentation) for detailed instructions on creating exams programmatically.

## API Documentation

The backend exposes a RESTful API for managing exams, questions, and responses. Below are the key endpoints, with a focus on dynamic question creation.

### POST /api/exams/

**Description**: Creates a new exam along with its questions in a single request. The client provides exam details and a list of questions, and the backend assigns the exam ID to the questions automatically.

**Request Payload**:
```json
{
  "title": "Math Exam",
  "description": "Basic math test",
  "skin": "math",
  "questions": [
    {
      "question_type": "MCQ",
      "content": { "text": "What is 2 + 2?" },
      "max_score": 5,
      "order": 1,
      "options": [
        { "content": { "text": "4" }, "is_correct": true },
        { "content": { "text": "5" }, "is_correct": false }
      ]
    },
    {
      "question_type": "TEXT",
      "content": { "text": "Explain addition." },
      "max_score": 10,
      "order": 2
    }
  ]
}
```

**Response**:
- **201 Created**: Returns the created exam with its ID and associated questions.
```json
{
  "id": 1,
  "title": "Math Exam",
  "description": "Basic math test",
  "skin": "math",
  "questions": [
    {
      "id": 1,
      "question_type": "MCQ",
      "content": { "text": "What is 2 + 2?" },
      "max_score": 5,
      "order": 1,
      "options": [
        { "id": 1, "content": { "text": "4" }, "is_correct": true },
        { "id": 2, "content": { "text": "5" }, "is_correct": false }
      ]
    },
    {
      "id": 2,
      "question_type": "TEXT",
      "content": { "text": "Explain addition." },
      "max_score": 10,
      "order": 2
    }
  ]
}
```

**Note**: The client does not need to specify an `exam_id` for questions. The backend creates the exam, generates an `exam_id`, and associates it with the questions during creation. For MCQ questions, options are also linked automatically.

### Other Endpoints

- **GET /api/exams/**: Lists all exams.
- **GET /api/exams/{exam_id}/**: Retrieves details of a specific exam, including its questions.
- **POST /api/responses/**: Submits a user’s responses to exam questions.
- **GET /api/users/{user_id}/responses/**: Retrieves all responses for a user.

**Detailed Docs**: Access `/api/docs/` when the server is running for a full API specification.

## Database Schema

The application uses a relational database with the following key tables. The design supports dynamic question creation while maintaining referential integrity.

- **Exams**:
  - `exam_id` (Primary Key)
  - `title` (String)
  - `description` (Text, optional)
  - `skin` (String, e.g., "math", "language")
  - `created_at` (DateTime)
  - `updated_at` (DateTime)

- **Questions**:
  - `question_id` (Primary Key)
  - `exam` (Foreign Key to Exams, assigned by backend)
  - `question_type` (Enum: "MCQ", "FILL_IN_THE_BLANK", "TEXT")
  - `content` (JSONField: stores text, images, etc.)
  - `max_score` (Integer)
  - `order` (Integer)

- **Options** (for MCQ questions):
  - `option_id` (Primary Key)
  - `question` (Foreign Key to Questions, assigned by backend)
  - `content` (JSONField: stores option text or media)
  - `is_correct` (Boolean)
  - `order` (Integer)

- **Blanks** (for fill-in-the-blank questions):
  - `blank_id` (Primary Key)
  - `question` (Foreign Key to Questions)
  - `correct_answer` (String)
  - `position` (Integer)

- **UserResponses**:
  - `response_id` (Primary Key)
  - `user` (Foreign Key to Users)
  - `question` (Foreign Key to Questions)
  - `answer` (JSONField: stores user input)
  - `submitted_at` (DateTime)
  - `score` (Integer, optional)

**Note**: When creating an exam with questions via the API, the `exam` field in the Questions table (and related fields in Options/Blanks) is populated automatically by the backend, ensuring a seamless creation process without requiring the client to manage IDs.

## Contribution Guidelines

We welcome contributions to enhance ExamApp! To get started:

1. **Fork the Repository**: Create your own copy on GitHub.
2. **Create a Branch**: Use a descriptive name, e.g., `feature/add-question-type`.
3. **Make Changes**: Implement your feature or fix, adhering to coding standards.
4. **Write Tests**: Ensure your changes are covered by tests.
5. **Submit a Pull Request**: Provide a clear description of your changes.

### Coding Standards

- **Backend**: Follow PEP 8 for Python code.
- **Frontend**: Use ESLint for JavaScript code consistency.
- Write clear, concise commit messages (e.g., "Add support for dynamic question creation").

### Running Tests

- **Backend**: `python manage.py test`
- **Frontend**: `npm test`

## License

This project is licensed under the Apache License 2.0. See the [LICENSE](LICENSE) file for details.
```
