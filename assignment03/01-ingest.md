## iot-sensor-1
-เป็น sensor ที่จำลองค่าที่อยู่ภายในเครื่อง server 

## iot-sensor-2
-เป็น sensor ที่จำลองค่าที่อยู่ภายในเครื่องคอมของคนในกลุ่ม

## iot-sensor-3
-เป็น sensor ที่รับค่าจริงจาก borad cucumber จากของกลุ่มตัวเอง

## iot-sensor-4-10
-เป็น sensor ที่รับค่าจริงจาก borad cucumber จากของกลุ่มเพื่อนทั้ง 7 กลุ่ม


# MQTT
MQTT (Message Queuing Telemetry Transport) เป็นโปรโตคอลการสื่อสารที่ออกแบบมาเพื่อการส่งข้อมูลระหว่างอุปกรณ์ในระบบ IoT (Internet of Things) โดยเฉพาะ มันเป็นโปรโตคอลที่เบาและมีประสิทธิภาพสูง ซึ่งเหมาะสำหรับการสื่อสารในสภาพแวดล้อมที่มีแบนด์วิธจำกัดหรือการเชื่อมต่อไม่เสถียร เช่น อุปกรณ์ที่ใช้แบตเตอรี่หรือในพื้นที่ห่างไกล <br>

# คุณสมบัติหลักของ MQTT:
1.Lightweight: การออกแบบที่เบา ทำให้ประหยัดพลังงานและแบนด์วิธ<br>
2.Publish/Subscribe Model: ใช้รูปแบบการสื่อสารที่ไม่ต้องมีการเชื่อมต่อโดยตรงระหว่างอุปกรณ์ โดยมี "Broker" เป็นตัวกลางในการรับและส่งข้อความ<br>
3.QoS (Quality of Service): มีการจัดการคุณภาพของการส่งข้อมูล เพื่อรับประกันว่าข้อมูลจะถูกส่งถึงปลายทางในระดับที่ต้องการ<br>
4.Retained Messages: สามารถเก็บข้อความล่าสุดไว้เพื่อให้ผู้สมัครรับข้อมูลใหม่สามารถรับข้อมูลได้ทันที<br>
5.Last Will and Testament: ฟีเจอร์ที่ทำให้อุปกรณ์สามารถส่งข้อความสุดท้ายก่อนจะตัดการเชื่อมต่อออกไป<br>

# การเชื่อมต่อระหว่าง MQTT Broker กับ Kafka
1.MQTT Broker รับข้อมูลจากอุปกรณ์ IoT ที่เชื่อมต่อผ่านโปรโตคอล MQTT<br>
2.MQTT Broker ส่งข้อมูลไปยัง Kafka Bridge<br>
3.Kafka Bridge ส่งข้อมูลเข้าสู่ Kafka Cluster โดยกำหนด Topic ที่จะเก็บข้อมูล<br>
4.ระบบ downstream หรือ application อื่น ๆ ที่เชื่อมต่อกับ Kafka จะสามารถดึงข้อมูลไปประมวลผลต่อได้<br>
