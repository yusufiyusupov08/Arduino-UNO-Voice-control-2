    #include <Servo.h>
    #include <Stepper.h>
    #include <SoftwareSerial.h>
    
    Servo myServo;  
    
    const int servoPin = 5;        // Сервопривод (SG90)
    const int ledPin = 6;          // Светодиод
    const int micRx = 7;           // RX микрофона
    const int micTx = 8;           // TX микрофона
    
    SoftwareSerial voiceModule(micRx, micTx);
    
    bool doorOpen = false;  
    bool lightOn = false;   
    
    // Настройки шагового двигателя
    const int stepsPerRevolution = 2048;  
    Stepper myStepper(stepsPerRevolution, 9, 10, 11, 12);  // Подключение к ULN2003
    
    bool stepperPosition = false;
    
    void setup() {
        pinMode(ledPin, OUTPUT);
        
    myServo.attach(servoPin);
    myServo.write(0);  
    digitalWrite(ledPin, LOW);

    myStepper.setSpeed(10);
    
    voiceModule.begin(9600);
    Serial.begin(9600);  
    Serial.println("Готово! Введите команду или используйте голосовые команды:");
    Serial.println("'d' - открыть/закрыть дверь");
    Serial.println("'l' - включить/выключить лампочку");
    Serial.println("'r' - шаговый двигатель вперед (270°)");
    Serial.println("'b' - шаговый двигатель назад (270°)");
    Serial.println("Голосовые команды: 'open the door', 'close the door', 'turn on the light', 'turn off the light', 'open the garage', 'close the garage'");
    }
    
    void loop() {
        if (Serial.available() > 0) {
            char command = Serial.read();  

        if (command == 'd') {  
            doorOpen = !doorOpen;  
            myServo.write(doorOpen ? 105 : 0);  
            Serial.println(doorOpen ? "Дверь открыта" : "Дверь закрыта");
        } 

        else if (command == 'l') {  
            lightOn = !lightOn;
            digitalWrite(ledPin, lightOn ? HIGH : LOW);
            Serial.println(lightOn ? "Лампочка включена" : "Лампочка выключена");
        } 

        else if (command == 'r') {  
            myStepper.step(stepsPerRevolution * 3 / 4);  
            Serial.println("Шаговый двигатель: вперед на 270°");
        } 

        else if (command == 'b') {  
            myStepper.step(-stepsPerRevolution * 3 / 4);  
            Serial.println("Шаговый двигатель: назад на 270°");
        } 

        else {  
            Serial.println("Ошибка: неизвестная команда");
        }
    }
    
    if (voiceModule.available() > 0) {
        String voiceCommand = voiceModule.readStringUntil('\n');
        voiceCommand.trim();
        
        if (voiceCommand == "open the door") {
            myServo.write(105);
            Serial.println("Голосовая команда: дверь открыта");
        } else if (voiceCommand == "close the door") {
            myServo.write(0);
            Serial.println("Голосовая команда: дверь закрыта");
        } else if (voiceCommand == "turn on the light") {
            digitalWrite(ledPin, HIGH);
            Serial.println("Голосовая команда: лампочка включена");
        } else if (voiceCommand == "turn off the light") {
            digitalWrite(ledPin, LOW);
            Serial.println("Голосовая команда: лампочка выключена");
        } else if (voiceCommand == "open the garage") {
            myStepper.step(-stepsPerRevolution * 5 / 4);
            Serial.println("Голосовая команда: шаговый двигатель назад на 320°");
        } else if (voiceCommand == "close the garage") {
            myStepper.step(stepsPerRevolution * 5 / 4);
            Serial.println("Голосовая команда: шаговый двигатель вперед на 320°");
        } else {
            Serial.println("Голосовая команда не распознана");
        }
    }
    }
    
    
    // "open the door" → Открывает дверь
    // "close the door" → Закрывает дверь
    // "turn on the light" → Включает свет
    // "turn off the light" → Выключает свет
    // "open the garage" → Поворачивает шаговый двигатель назад на 320°
    // "close the garage" → Поворачивает шаговый двигатель вперед на 320°
    # Arduino-UNO-Voice-control-2
