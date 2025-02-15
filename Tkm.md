## Main Bot File (main.js)
```js
// Import the necessary modules.
const { Client, LocalAuth, MessageMedia } = require('whatsapp-web.js');
const express = require('express');
const qrcode = require('qrcode');
const fs = require('fs');
const { body, validationResult } = require('express-validator');

// Create a WhatsApp client.
const client = new Client({
  authStrategy: new LocalAuth(),
  puppeteer: {
    headless: true,
  },
});

// Start the WhatsApp client.
client.on('qr', (qr) => {
  console.log('QR RECEIVED', qr);
  qrcode.toFile('./qr.png', qr, (err) => {
    if (err) throw err
    console.log('QR saved to ./qr.png');
  })
});

client.on('ready', () => {
  console.log('Client is ready!');
});

client.on('message', (msg) => {
 const text = msg.body.toLowerCase()
  if (text.includes('hi') || text.includes('hello')) {
    msg.reply('Hello there!');
  }
});

client.initialize();

// Create an Express app.
const app = express();

// Use the Express body parser.
app.use(express.json());

// Define a POST route to handle incoming messages.
app.post('/message', [
  // Validate the request body.
  body('message').exists().isString(),
  body('number').exists().isMobilePhone(),
], (req, res) => {
  // Get the validation result.
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }

  // Get the message and number from the request body.
  const { message, number } = req.body;

  // Send a WhatsApp message to the specified number.
  client.sendMessage(number + '@c.us', message).then((response) => {
    // Send a success response to the client.
    res.status(200).json({ message: 'Message sent successfully' });
  }).catch((error) => {
    // Send an error response to the client.
    res.status(500).json({ error: error.toString() });
  });
});

// Start the Express app.
app.listen(3000, () => {
  console.log('Express app is listening on port 3000');
});
```

## package.json
```
{
  "name": "whatsapp-bot",
  "version": "1.0.0",
  "description": "A basic WhatsApp bot using whatsapp-web.js",
  "main": "main.js",
  "scripts": {
    "start": "node main.js",
    "dev": "nodemon main.js"
  },
  "dependencies": {
    "express": "^4.17.1",
    "express-validator": "^6.12.3",
    "qrcode": "^1.5.1",
    "whatsapp-web.js": "^2.0.5"
  },
  "devDependencies": {
    "nodemon": "^2.0.15"
  }
}
```

## .env
```
PORT=3000
```

#### QR code:

![QR code](qr.png)

#### Usage:

1. Clone the repository.
2. Install the dependencies.
3. Run the bot.
4. Open WhatsApp on your phone and scan the QR code.
5. Once the bot is connected, you can send messages to the specified number using the `/message` route.

#### Documentation:

* [whatsapp-web.js documentation](https://docs.whatsapp.com/en/web/)
* [Express documentation](https://expressjs.com/en/api.html)
* [qrcode documentation](https://www.npmjs.com/package/qrcode)
* [express-validator documentation](https://express-validator.github.io/docs/)

#### Implementation details:

* The bot uses the `whatsapp-web.js` library to connect to WhatsApp.
* The bot uses the `express` framework to handle incoming HTTP requests.
* The bot uses the `qrcode` library to generate a QR code for the user to scan.
* The bot uses the `express-validator` library to validate the request body.

#### Notes:

* The bot is still in development and may not be fully functional.
* The bot requires a WhatsApp account to be connected to.
* The bot can only send messages to numbers that are in the user's address book.

#### Feedback:

If you have any feedback or questions, please feel free to create an issue on the GitHub repository.