# ExamApp

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

## Table of Contents
1. [Introduction](#introduction)
2. [Key Features](#key-features)
3. [System Architecture](#system-architecture)
4. [Workflow](#workflow)
5. [Database Schema (Django Models)](#database-schema-django-models)
6. [API Endpoints (Django REST Framework)](#api-endpoints-django-rest-framework)
7. [Frontend (React)](#frontend-react)
8. [Implementation Notes](#implementation-notes)
9. [Contribution Guidelines](#contribution-guidelines)
10. [License](#license)

---

## Introduction
ExamApp is a web-based platform for creating, managing, and taking exams, quizzes, and surveys. It supports fetching questions from an external API, displaying rich content (text, images, code, math formulas) in questions and options, and validating answers on the backend. It includes features like customizable UI skins, timed assessments, progress tracking, and performance reporting, with distinct roles for students and instructors.

## Key Features
- **Frontend**: Built with **React** for dynamic, component-based UI rendering.
- **Backend**: Built with **Django** for robust API management and database handling.
- **Question Support**: Local questions stored in the database and external questions fetched from an API.
- **Rich Content**: Text, images, code, and math formulas in questions and options.
- **Authentication**: Secure login for students and instructors.
- **Instructor Tools**: Create questions, solutions, and assessments.
- **Student Tools**: Take assessments, track progress, view reports.
- **Timing**: Enforce time limits for assessments and sections.
- **Skins**: Customizable UI themes for different exam types.

---

## System Architecture
The application follows a client-server model:
- **Frontend (React)**: Handles UI rendering, user interactions, timers, and progress tracking.
- **Backend (Django)**: Manages APIs, database operations, authentication, and external API integration.
- **Database**: Relational database (e.g., PostgreSQL) for storing local data.
- **External API**: Provides additional questions and answer validation.
- **Media Storage**: Stores images (e.g., Django media folder or AWS S3).

## Workflow
### Student Flow
1. Logs in via React frontend.
2. Selects an assessment; React fetches data from Django API.
3. Displays questions (local or external) with rich content.
4. Submits answers; Django validates and stores results.
5. Views performance report.

### Instructor Flow
1. Logs in and creates questions/assessments via React forms.
2. Selects local or external questions; Django saves to database.
3. Reviews student reports.

---

## Database Schema (Django Models)
Django's ORM defines the database structure with models distinguishing between local and external questions.

### Models
#### 1. User
Extends Django's AbstractUser for role-based access.
```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    ROLE_CHOICES = (
        ('student', 'Student'),
        ('instructor', 'Instructor'),
    )
    role = models.CharField(max_length=10, choices=ROLE_CHOICES)
```

#### 2. Category
Groups questions by topic.
```python
class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)
    description = models.TextField(blank=True)
```

#### 3. Question (Local Only)
Stores locally created questions.
```python
class Question(models.Model):
    content = models.TextField()  # HTML/Markdown with media URLs
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    created_by = models.ForeignKey(User, on_delete=models.CASCADE, limit_choices_to={'role': 'instructor'})
    has_correct_answer = models.BooleanField(default=True)
    solution = models.TextField(blank=True)
```

#### 4. Option (Local Only)
Options for local questions.
```python
class Option(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE, related_name='options')
    content = models.TextField()  # HTML/Markdown with media URLs
    is_correct = models.BooleanField()
```

#### 5. Assessment
Represents exams, quizzes, or surveys.
```python
class Assessment(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    type = models.CharField(max_length=10, choices=[('exam', 'Exam'), ('quiz', 'Quiz'), ('survey', 'Survey')])
    time_limit = models.IntegerField(null=True, blank=True)  # in seconds
    skin = models.CharField(max_length=50)
    created_by = models.ForeignKey(User, on_delete=models.CASCADE, limit_choices_to={'role': 'instructor'})
    start_time = models.DateTimeField(null=True, blank=True)
    end_time = models.DateTimeField(null=True, blank=True)
```

#### 6. StudentAssessment
Tracks student progress.
```python
class StudentAssessment(models.Model):
    student = models.ForeignKey(User, on_delete=models.CASCADE, limit_choices_to={'role': 'student'})
    assessment = models.ForeignKey(Assessment, on_delete=models.CASCADE)
    start_time = models.DateTimeField(auto_now_add=True)
    end_time = models.DateTimeField(null=True, blank=True)
    status = models.CharField(max_length=10, choices=[('started', 'Started'), ('completed', 'Completed')])
    score = models.IntegerField(null=True, blank=True)
```

#### 7. Answer
Stores student responses.
```python
class Answer(models.Model):
    student_assessment = models.ForeignKey(StudentAssessment, on_delete=models.CASCADE, related_name='answers')
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    option = models.ForeignKey(Option, on_delete=models.CASCADE)
    is_correct = models.BooleanField(null=True, blank=True)
    submitted_at = models.DateTimeField(auto_now_add=True)
```

---

## API Endpoints (Django REST Framework)
### Authentication
- **POST /api/auth/login/**  
  **Body**: `{ "username": "string", "password": "string" }`  
  **Response**: `{ "token": "string", "user": { "id": int, "username": "string", "role": "string" } }`

- **GET /api/users/me/**  
  **Headers**: `Authorization: Bearer <token>`  
  **Response**: `{ "id": int, "username": "string", "role": "string" }`

### Questions API
- **GET /api/questions/** – Fetch all questions
- **POST /api/questions/** – Create a new question
- **GET /api/questions/{id}/** – Fetch a single question
- **PUT /api/questions/{id}/** – Update a question
- **DELETE /api/questions/{id}/** – Delete a question

### Options API
- **GET /api/questions/{id}/options/** – Fetch all options for a question
- **POST /api/questions/{id}/options/** – Create an option for a question
- **PUT /api/options/{id}/** – Update an option
- **DELETE /api/options/{id}/** – Delete an option

### Assessments API
- **GET /api/assessments/** – Fetch all assessments
- **POST /api/assessments/** – Create a new assessment
- **GET /api/assessments/{id}/** – Fetch a single assessment
- **PUT /api/assessments/{id}/** – Update an assessment
- **DELETE /api/assessments/{id}/** – Delete an assessment

### Student Assessments API
- **GET /api/student-assessments/** – Fetch all student assessments
- **POST /api/student-assessments/** – Start an assessment
- **GET /api/student-assessments/{id}/** – Fetch a student's assessment
- **PUT /api/student-assessments/{id}/** – Update student assessment status

### Answers API
- **POST /api/answers/** – Submit an answer
- **GET /api/answers/{id}/** – Fetch a student’s answer

---

## Frontend (React)
The React frontend is component-based, leveraging libraries for rich content and interactivity.

### Components
- **QuestionDisplay**: Renders questions/options with:
  - `react-markdown` for text.
  - `react-mathjax` for math formulas.
  - `react-syntax-highlighter` for code.
  - `<img>` tags for images.
- **Timer**: Uses `useState` and `setInterval` for countdowns.
- **ProgressBar**: Tracks answered questions in state.
- **AssessmentForm**: Instructor form for creating assessments/questions.

---

## Implementation Notes
### Backend
- **DRF Authentication**: Uses `TokenAuthentication` for secure API access.
- **External API**: Batch fetch external questions if supported.
- **Validation**: Local questions check `Option.is_correct`; external questions proxy to API.

### Frontend
- **State Management**: Uses Redux or Context API for complex state.
- **Security**: Sanitizes HTML with DOMPurify to prevent XSS.

---

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

---

## License

This project is licensed under the Apache License 2.0. See the [LICENSE](LICENSE) file for details.
```
