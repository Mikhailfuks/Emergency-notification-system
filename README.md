// index.js
const express = require('express');
const nodemailer = require('nodemailer');
const bodyParser = require('body-parser');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// Create a transporter for sending emails
const transporter = nodemailer.createTransport({
    host: process.env.SMTP_HOST,
    port: process.env.SMTP_PORT,
    secure: false, // set to true if using port 465
    auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS,
    },
});

// Route to send emergency notifications
app.post('/send-notification', async (req, res) => {
    const { subject, message, recipients } = req.body;

    const mailOptions = {
        from: process.env.SMTP_USER, // sender address
        to: recipients.join(', '),    // list of receivers
        subject: subject,              // Subject line
        text: message,                 // plain text body
        html: `<p>${message}</p>`,     // html body
    };

    try {
        const info = await transporter.sendMail(mailOptions);
        console.log('Message sent: %s', info.messageId);
        res.status(200).json({ message: 'Notification sent successfully!', messageId: info.messageId });
    } catch (error) {
        console.error('Error sending email:', error);
        res.status(500).json({ message: 'Failed to send notification', error: error.message });
    }
});

// Start the server
app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});

curl -X POST http://localhost:3000/send-notification \
-H "Content-Type: application/json" \
-d '{
    "subject": "Emergency Alert",


    "message": "This is a test notification for an emergency situation.",
    "recipients": ["recipient1@example.com", "recipient2@example.com"]
}'
