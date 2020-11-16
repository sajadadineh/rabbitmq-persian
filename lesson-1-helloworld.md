# send.js

![image](https://www.rabbitmq.com/img/tutorials/sending.png)

ما به دوتا برنامه نیاز داریم که یکی پیامی رو تو صف بزاره به اصطلاح ارسال کننده ی پیام باشه و دومی دریافت کننده پیام از داخل صف باشه

پس اول من برنامه ارسال کننده رو مینویسم و بعد دریافت کننده البته فرقی نداره و میشه اول برنامه ی دریافت کننده ی پیام رو نوشت


### مرحله اول

اول باید کتابخونه رابیت ام کیو رو نصب و اضافه کنیم

```javascript
npm install amqplib
```

```javascript
const amqp = require('amqplib/callback_api');
```

### مرحله دوم

بعد از نصب کتابخونه باید به خود سرور رابیت ام کیو کانکشن برقرار کنید من چون رابیت روتو کامپیوتر خودم دارم وصل میشم به لوکال هاست خودم و بعد یک 
کانال درست میکنم تا بتونم تو اون کانال صف هامو درست کنم و بتونم پیام ردو بدل کنم

```javascript
amqp.connect('amqp://localhost', (connError, connection) => {
  if(connError){
    throw connError;
  }
  connection.createChannel((channelError, channel) => {
    if(channelError){
      throw channelError;
    }
  });
})
``` 

### مرحله سوم

حالا باید صف سلامم رو درست کنم تا بتونم پیامام رو تو این صف بزارم
این متد یک اپشنی میگیره که وقتی اگه سرور رابیتمون داون شد یا برنامش کرش کرد و براش مشکلی پیش اومد صف رو تو هارد ذخیره کنه تا صفمون از بین نره دقت داشته باشین که این اپشن دوریبل فقط صف رو ذخیره میکنه 
که در حال حاظر هم فالس هست و ما نیازی فعلا به این اپشن نداریم


```javascript
channel.assertQueue("hello", {durable: false});
```

### مرحله چهارم

خب حالا میخوایم پیاممون رو ارسال کنیم

این متد اول اسم یک صف و پیام ما رو میگیره و تو صف ارسال میکنه


```javascript
const msg = "hello world"
channel.sendToQueue("hello", Buffer.from(msg));
console.log(" [x] Sent %s", msg);
``` 

### کد برنامه 

الان برنامه ی ارسال پیاممون کامل شده و میتونید رانش کنیم تا پیامی رو تو صف قرار بده
و در آخر هم باید کانکشن رو قطع کنیم برای همین من در آخر برنامه یه تیکه کدی نوشتم تا بعد از گذاشتن پیام در صف برنامه بسته بشه


```javascript
const amqp = require('amqplib/callback_api');

amqp.connect('amqp://localhost', (connError, connection) => {
  if(connError){
    throw connError;
  }
  connection.createChannel((channelError, channel) => {
  
    if(channelError){
      throw channelError;
    }
    
    const queue = "hello"
    const msg = "hello world"
    
    channel.assertQueue(queue, {durable: false});
    
    channel.sendToQueue(queue, Buffer.from(msg));
    console.log(" [x] Sent %s", msg);
  });
  setTimeout(function() {
        connection.close();
        process.exit(0);
    }, 500);
})
```

# receive.js

![image](https://www.rabbitmq.com/img/tutorials/receiving.png)

الان باید دریافت کننده ی پیاممون رو بنویسیم


### مرحله ی اول

خب کدومون تا مرحله ی سوم کد ارسال کننده یکیه و شاید براتون سوال بشه که چرا باز تو این برنامه هم ما مرحله ی سوم رو دوباره تکرار میکنیم و دوباره یک صف درست میکنیم

اول اینکه ما نمیدونیم قراره کدوم برنامه زودتر اجرا بشه شاید دریافت کننده ی پیام زودتر اجرا بشه پس برای همین ما در هر دو برنامه صف رو درست میکنیم ولی اگه برای مثال 

اول ارسال کننده ی پیام رو اجرا کنیم یک صف تشکیل میشه و وقتی دریافت کننده ی پیام رو اجرا کنیم دیگه از اون قسمت که دوباره صف رو درست میکنیم رد میشه

پس تیکه ی اول کدومون تو این فایل هم به این شکل میشه

```javascript
const amqp = require('amqplib/callback_api');

amqp.connect('amqp://localhost', (connError, connection) => {
  if(connError){
    throw connError;
  }
  connection.createChannel((channelError, channel) => {
  
    if(channelError){
      throw channelError;
    }
    
    const queue = "hello"

    channel.assertQueue(queue, {durable: false});

  });
})
```

### مرحله ی دوم 

حال باید متد کانسیوم رو صدا بزنیم و اول اسم صفمون رو بهش بدیم و بعد یک فانکشن که وقتی پیاممون اومد چه کاری انجام بشه که در اینجا فقط پیام رو لاگ میکنه و این متد اپشنی میگیره به اسم اکت که کارش اینه وقتی پیام رو به مصرف کننده میده اون پیام رو از صف حذف میکنه
که ما میتونیم این اکت رو فالس کنیم تا خودمون بصورت دستی اکت رو وارد کنیم که این پیامی که از صف گرفتیم الان کارش تموم شد و بهش اکت برگردونیم ولی در اینجا نیاز نیست و در درس های بعدی به این اپشن بیشتر میپردازیم


```javascript
console.log(" [*] Waiting for messages in %s. To exit press CTRL+C", queue);
channel.consume(queue, (msg) => {
  console.log(" [x] Received %s", msg.content.toString());
}, {noAck: true});
```

 ### کد برنامه 

```javascript
const amqp = require('amqplib/callback_api');

amqp.connect('amqp://localhost', (connError, connection) => {
  if(connError){
    throw connError;
  }
  connection.createChannel((channelError, channel) => {
  
    if(channelError){
      throw channelError;
    }
    
    const queue = "hello"

    channel.assertQueue(queue, {durable: false});
    
    console.log(" [*] Waiting for messages in %s. To exit press CTRL+C", queue);
    channel.consume(queue, (msg) => {console.log(" [x] Received %s", msg.content.toString())}, {noAck: true});
  });
})
```

خب حالا میتونید برنامه رو ران کنید و میبینید که پیام ارسال میشه و از برنامه دریافت کننده مون هم دریافت میشه

```
node send.js

node receive.js
```

شما میتونید حتی رو مرورگرتون به [لوکال هاستتون](https://localhost:15672) به پورت 15672 یا پورت 5672 وصل شید و با یوزرنیم پسورد دیفالت(۱) به پنل رابیت ام کیو خودتون برید و بتونید اطلاعات بیشتری رو دریافت کنید

در [درس بعدی](https://github.com/sajadadineh/rabbitmq-persian/blob/main/lesson-2-work-queues.md) ما چندین دریافت کننده ی پیام درست میکنیم و چندین پیام رو داخل صف میزاریم تا دقیق تر ببینیم رابیت ام کیو چگونه کار میکنه
و بیشتر کار عملی و واقعی انجام میدیم و مفاهیم صف بندی با الگریتم نوبت گردشی رو متوجه میشیم

---

(۱) username:guest  ,   password:guest
