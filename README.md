#include <WiFi.h>
#include <ESP_Mail_Client.h>

#define PIR_PIN 13

// WiFi
#define WIFI_SSID "Airtel_moha_4524"
#define WIFI_PASSWORD "Air@09640"

// Gmail SMTP
#define SMTP_HOST "smtp.gmail.com"
#define SMTP_PORT 465

#define AUTHOR_EMAIL "agentakram007@gmail.com"
#define AUTHOR_PASSWORD "vomnvrrdyeegoehi"

#define RECIPIENT_EMAIL "akrammohammad0705@gmail.com"

SMTPSession smtp;

bool motionSent = false;

void setup() {
  Serial.begin(115200);
  pinMode(PIR_PIN, INPUT);

  // Connect WiFi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nConnected!");

  delay(30000); // PIR warm-up
}

void sendEmail() {
  SMTP_Message message;

  message.sender.name = "ESP32 Alert";
  message.sender.email = AUTHOR_EMAIL;
  message.subject = "🚨 Motion Detected!";
  message.addRecipient("User", RECIPIENT_EMAIL);

  message.text.content = "Motion detected by PIR sensor!";
  message.text.charSet = "utf-8";

  ESP_Mail_Session session;
  session.server.host_name = SMTP_HOST;
  session.server.port = SMTP_PORT;
  session.login.email = AUTHOR_EMAIL;
  session.login.password = AUTHOR_PASSWORD;

  if (!smtp.connect(&session)) {
    Serial.println("SMTP connection failed");
    return;
  }

  if (!MailClient.sendMail(&smtp, &message)) {
    Serial.println("Error sending Email");
  } else {
    Serial.println("Email Sent Successfully!");
  }

  smtp.closeSession();
}

void loop() {
  int motion = digitalRead(PIR_PIN);

  if (motion == HIGH && !motionSent) {
    Serial.println("Motion Detected! Sending Email...");
    sendEmail();
    motionSent = true;
  }

  if (motion == LOW) {
    motionSent = false;
  }

  delay(500);
}
