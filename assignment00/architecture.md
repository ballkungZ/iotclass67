# Main technologies of architecture

## Architecture Overview

![1](https://github.com/user-attachments/assets/961c48ad-c6c7-4cd7-b4fd-98f3c52b9add)

## Eclipse Mosquitto
เป็นโปรเจคโอเพนซอร์สที่ให้บริการ Broker MQTT ซึ่งเป็นโปรแกรมที่ทำหน้าที่เป็นตัวกลางในการสื่อสารระหว่างอุปกรณ์ IoT หรืออุปกรณ์อื่น ๆ กับโครงข่ายอินเทอร์เน็ตและ Mosquito มีน้ำหนักที่เบาและสามารถใช้ได้ทุกอุปกรณ์ โดยที่อุปกรณ์ที่เป็น Publisher สามารถส่งข้อมูลไปยัง Broker และอุปกรณ์ที่เป็น Subscriber สามารถรับข้อมูลจาก Broker ได้ 


## Apache ZooKeeper
เป็นโอเพนซอร์สที่เป็นระบบบริการการจัดการข้อมูลแบบแตกต่างๆ ในระบบคอมพิวเตอร์แบบกระจายโดย Zookeeper เป็นระบบบริการที่ให้บริการการจัดการและเก็บข้อมูลแบบโครงสร้าง ถ้านำไปใช้กับแอปพลิเคชั่นจะมีความซับซ้อน


## Apache Kafka
เป็นแพลตฟอร์มการรับส่งข้อมูลแบบกระจายที่ช่วยให้คุณสามารถเผยแพร่ จัดเก็บ และประมวลผลสตรีมบันทึก และสมัครรับข้อมูลแบบเรียลไทม์ ได้รับการออกแบบมาเพื่อจัดการสตรีมข้อมูลจากแหล่งต่างๆ แจกจ่ายให้กับผู้ใช้ต่างๆ หรือก็คือ จะถ่ายโอนข้อมูลจำนวนมหาศาล ไปจุดต่างๆที่คุณต้องการ ได้พร้อมกันทั้งหมดนี้ในเวลาเดียวกัน 


## Apache Kafka Connect
ซึ่งเป็นส่วนประกอบโอเพ่นซอร์สของ Apache Kafka เป็นเฟรมเวิร์กสำหรับการเชื่อมต่อ Kafka กับระบบภายนอก เช่น ฐานข้อมูล การจัดเก็บ Key - value ค่า indexในการค้นหา และระบบไฟล์ โดยKafka Connect มุ่งเน้นไปที่การสตรีมข้อมูลเข้าและออกจาก Kafka ทำให้คุณเขียนปลั๊กอินตัวเชื่อมต่อคุณภาพสูง เชื่อถือได้ และประสิทธิภาพสูงได้ง่ายขึ้น นอกจากนี้ยังช่วยให้กรอบงานสามารถรับประกันที่ยากต่อการบรรลุผลโดยใช้กรอบงานอื่น Kafka Connect เป็นองค์ประกอบสำคัญของในบทความนี้ได้อธิบายการใช้ Kafka Connect APIs และ Kafka Streams API ในการสร้าง ETL pipeline เพื่อจัดการกับข้อมูล IoT และเซ็นเซอร์ขนาดใหญ่ ซึ่งประกอบไปด้วยการนำเข้าข้อมูล (Extract), การแปลงข้อมูล (Transform), และการโหลดข้อมูล (Load) เพื่อรองรับการตรวจสอบและประมวลผลข้อมูลแบบเรียลไทม์ โดยใช้เทคโนโลยีหลักหลายอย่าง เช่น Apache Kafka, MongoDB, และ Prometheus เพื่อสร้างสถาปัตยกรรมการสตรีมข้อมูลที่มีประสิทธิภาพสูงและสามารถขยายตัวได้ง่าย 


## Apache Kafka Streams
คือไลบรารี client สำหรับสร้างแอปพลิเคชันและไมโครเซอร์วิส โดยที่ข้อมูลอินพุตและเอาต์พุตจะถูกจัดเก็บไว้ในคลัสเตอร์ Apache Kafka เป็นการผสมผสานความเรียบง่ายของการเขียนและการปรับใช้แอปพลิเคชัน Java และ Scala มาตรฐานบนฝั่งไคลเอ็นต์เข้ากับประโยชน์ของเทคโนโลยีคลัสเตอร์ฝั่งเซิร์ฟเวอร์ของ Kafka 


## Prometheus
เป็นแอปพลิเคชันซอฟต์แวร์ที่ใช้ในการติดตามและแจ้งเตือนเหตุการณ์ณ์ โดยจะบันทึกตัววัดแบบเรียลไทม์ในฐานข้อมูลอนุกรมเวลาที่สร้างขึ้นโดยใช้โมเดลการดึง HTTP พร้อมการสืบค้นที่ยืดหยุ่นและการแจ้งเตือนแบบเรียลไทม์  


## MongoDB
เป็นระบบฐานข้อมูล NoSQL แบบโอเพ่นซอร์ส เน้นเอกสาร แทนที่จะจัดเก็บข้อมูลในตาราง เช่นเดียวกับที่ทำในฐานข้อมูลเชิงสัมพันธ์ MongoDB จะบันทึกโครงสร้างข้อมูล BSON มาแบบไดนามิก ทำให้การรวมข้อมูลในแอปพลิเคชันบางตัวง่ายขึ้นและเร็วขึ้น MongoDB เป็นฐานข้อมูลที่เหมาะสำหรับใช้ในการผลิตและมีฟังก์ชันการทำงานที่หลากหลาย 


## Grafana
เป็นซอฟต์แวร์ฟรีที่ใช้ Apache 2.0 ใบอนุญาตที่อนุญาตให้แสดงและจัดรูปแบบข้อมูลเมตริก ช่วยให้คุณสร้างแดชบอร์ดและแผนภูมิจากหลายแหล่ง รวมถึงฐานข้อมูลอนุกรมเวลา เช่น Prometheus, Graphite, InfluxDB เดิมทีเริ่มต้นโดยเป็นส่วนหนึ่งของ Kibana และต่อมาได้แตกแขนงออกไป 

