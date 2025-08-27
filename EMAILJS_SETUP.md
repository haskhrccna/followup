# EmailJS Setup Guide for Automatic Weekly Reports

This guide will help you set up EmailJS to enable automatic weekly PDF report generation and email delivery to haskhr@hotmail.com.

## Step 1: Create EmailJS Account

1. Go to [EmailJS.com](https://www.emailjs.com/)
2. Sign up for a free account
3. Verify your email address

## Step 2: Create Email Service

1. In your EmailJS dashboard, go to "Email Services"
2. Click "Add New Service"
3. Choose your email provider (recommended: Gmail or Outlook)
4. Follow the setup instructions to connect your email account
5. Note down your **Service ID** (e.g., `service_abc123`)

## Step 3: Create Email Template

1. Go to "Email Templates" in your dashboard
2. Click "Create New Template"
3. Use this template structure:

### Template Content:
```
Subject: 📊 Weekly Exam Readiness Report - {{student_name}}

Dear Raghad,

Your weekly exam readiness report has been automatically generated.

Student: {{student_name}}
Report Date: {{report_date}}
Overall Readiness: {{overall_readiness}}%
Total Subjects: {{subject_count}}

The detailed PDF report is attached to this email.

This report was generated automatically every Saturday as requested.

Best regards,
Exam Readiness Tracker System
```

### Template Variables:
- `{{student_name}}` - Student name
- `{{report_date}}` - Report generation date
- `{{overall_readiness}}` - Overall readiness percentage
- `{{subject_count}}` - Number of subjects
- `{{pdf_attachment}}` - PDF file attachment

4. Set the template to send to: `haskhr@hotmail.com`
5. Set importance/priority to "High"
6. Note down your **Template ID** (e.g., `template_xyz789`)

## Step 4: Get Your Public Key

1. Go to "Account" → "General" in your EmailJS dashboard
2. Find your **Public Key** (e.g., `user_abcdef123456`)

## Step 5: Update Application Code

1. Open `index.html` in your GitHub repository
2. Find this line:
   ```javascript
   emailjs.init("YOUR_EMAILJS_PUBLIC_KEY");
   ```
3. Replace `YOUR_EMAILJS_PUBLIC_KEY` with your actual public key

4. Find these lines:
   ```javascript
   await emailjs.send(
       'YOUR_SERVICE_ID',
       'YOUR_TEMPLATE_ID',
       templateParams
   );
   ```
5. Replace:
   - `YOUR_SERVICE_ID` with your Service ID
   - `YOUR_TEMPLATE_ID` with your Template ID

## Step 6: Test the Setup

1. Deploy your updated application to GitHub Pages
2. Access the application on a Saturday between 8 AM - 10 PM
3. The system should automatically:
   - Show a notification about report generation
   - Generate a PDF report
   - Send it to haskhr@hotmail.com with high importance

## Step 7: Email Configuration for High Importance

To ensure emails are marked as high importance:

### For Gmail Service:
- In your EmailJS template, add custom headers:
  ```
  X-Priority: 1
  X-MSMail-Priority: High
  Importance: high
  ```

### For Outlook Service:
- The importance flag is automatically handled by the template parameters

## Troubleshooting

### Common Issues:

1. **Emails not sending:**
   - Check your EmailJS service is properly connected
   - Verify your email account has sufficient quota
   - Check spam/junk folders

2. **PDF not attaching:**
   - Ensure the PDF generation is working locally first
   - Check browser console for JavaScript errors
   - Verify base64 encoding is working

3. **Weekly schedule not working:**
   - The system requires the user to visit the site at least once on Saturday
   - Check browser notifications are enabled
   - Verify localStorage is working

### Testing Tips:

1. **Test Email Sending:**
   - Temporarily change the day check to current day for testing
   - Use browser developer tools to manually call `generateAndEmailReport()`

2. **Test PDF Generation:**
   - Click the "Full Report" button to ensure PDF generation works
   - Check browser downloads folder

3. **Test Notifications:**
   - Ensure browser notifications are enabled
   - Test with `showNotification()` function in console

## Security Notes

- Your EmailJS public key is safe to include in client-side code
- EmailJS handles authentication securely
- The email template prevents spam by using your verified email service
- Rate limiting is handled by EmailJS automatically

## Support

If you encounter issues:
1. Check the EmailJS dashboard for error logs
2. Review browser console for JavaScript errors
3. Test individual components (PDF generation, email sending) separately
4. Contact EmailJS support for service-specific issues

---

**Important:** Remember to update the three key values in your `index.html`:
1. `YOUR_EMAILJS_PUBLIC_KEY`
2. `YOUR_SERVICE_ID` 
3. `YOUR_TEMPLATE_ID`

Once configured, the system will automatically send weekly reports every Saturday!

