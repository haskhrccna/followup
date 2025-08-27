# Exam Readiness Tracker

A comprehensive exam readiness tracking application for students to monitor their study progress and exam performance across multiple subjects.

## Features

- **Subject Management**: Track progress across Arabic, English, Chemistry, Physics, and Biology
- **Study Progress**: Monitor chapter completion with date tracking
- **Exam Results**: Record exam scores with historical data and previous score tracking
- **PDF Reports**: Generate comprehensive performance reports
- **📧 Automatic Weekly Reports**: PDF reports automatically generated and emailed every Saturday
- **🔔 Smart Notifications**: Browser notifications for report generation and reminders
- **Secure Authentication**: Login system with session management
- **Responsive Design**: Works on desktop and mobile devices
- **Local Storage**: All data persists in browser localStorage

## Live Application

🌐 **Access the application**: [followup.hassan-adam.com](https://followup.hassan-adam.com)

## Login Credentials

- **Username**: Raghad
- **Password**: testreview

## Technology Stack

- **Frontend**: HTML5, CSS3, JavaScript (ES6+)
- **PDF Generation**: jsPDF library
- **Storage**: Browser localStorage
- **Hosting**: GitHub Pages
- **Domain**: Custom domain with subdirectory routing

## Features Overview

### 📊 Dashboard
- Overall readiness percentage across all subjects
- Individual subject cards with progress indicators
- Quick access to detailed subject views
- PDF report generation

### 📚 Subject Details
- Chapter-by-chapter study progress tracking
- Exam score recording with date stamps
- Previous score history display
- Real-time readiness calculations

### 📄 PDF Reports
- Comprehensive performance summaries
- Subject-by-subject breakdowns
- Date-stamped report generation
- Professional formatting

### 📧 Automatic Weekly Email Reports
- **Schedule**: Every Saturday between 8 AM - 10 PM
- **Recipient**: haskhr@hotmail.com
- **Priority**: High importance flag
- **Content**: Complete PDF report with all progress data
- **Notifications**: Browser alerts when reports are generated
- **Reminders**: Friday evening notifications to update progress

### 🔔 Smart Notifications
- Weekly report generation alerts
- Friday evening progress reminders
- Browser notification support
- Non-intrusive user experience

### 🔐 Authentication
- Secure login system
- Session persistence
- Automatic logout functionality

## Usage

1. **Login**: Use the provided credentials to access the application
2. **Dashboard**: View overall progress and navigate to specific subjects
3. **Study Tracking**: Mark chapters as completed in subject detail pages
4. **Exam Recording**: Enter exam scores and dates
5. **Report Generation**: Click "Full Report" to download PDF summaries
6. **Data Persistence**: All changes are automatically saved to localStorage

## Setup Instructions

### Basic Deployment
1. Upload all files to your GitHub repository
2. Enable GitHub Pages in repository settings
3. Configure custom domain: `followup.hassan-adam.com`
4. Set up DNS CNAME record pointing to your GitHub Pages URL

### Email Functionality Setup
To enable automatic weekly email reports:

1. **Create EmailJS Account**: Sign up at [EmailJS.com](https://www.emailjs.com/)
2. **Configure Email Service**: Connect your email provider (Gmail/Outlook recommended)
3. **Create Email Template**: Set up template for weekly reports
4. **Update Application**: Replace placeholder values in `index.html`:
   - `YOUR_EMAILJS_PUBLIC_KEY`
   - `YOUR_SERVICE_ID`
   - `YOUR_TEMPLATE_ID`

📋 **Detailed Setup Guide**: See `EMAILJS_SETUP.md` for complete instructions

### Browser Permissions
- **Notifications**: Allow browser notifications for report alerts
- **Downloads**: Ensure PDF downloads are enabled
- **LocalStorage**: Required for data persistence

## Development

This is a static web application designed for GitHub Pages deployment. All functionality is client-side with no backend dependencies.

### Local Development

```bash
# Serve the files locally
python3 -m http.server 8080

# Access at http://localhost:8080
```

## Student Information

**Student**: Raghad Hassan Adam  
**Application**: Exam Readiness Tracker  
**Domain**: followup.hassan-adam.com

---

*Built with ❤️ for academic success*

