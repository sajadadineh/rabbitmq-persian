<div dir='rtl'>
  
![image](https://www.rabbitmq.com/img/tutorials/python-two.png)

تو این مثال ما دو تا مصرف کننده داریم بزارید این مبحث رو با مثال شروع کنم

اگه مثال رستوران رو یادتون باشه گفته بودیم که اگه دوتا پیتزا تو صف سفارشات باشه و ما دوتا آشپز داشته باشیم یکی از پیتزا ها رو یکی از آشپز ها درست میکنه و اون یکی رو یک آشپز دیگه

تو این مثال هم به همین شکل هست رابیت ام کیو به نوبت به این ورکر(۱) یا همون دریافت کننده ی پیام ها رو ارسال میکنه 

به این روش نوبت گردشی(۲) گفته میشه

حالا بریم سراغ کد زدن تا بهتر بتونیم با این روش آشنا بشیم

ما در اینجا دوتا فایل داریم که نام یکی از اون ها رو تسک و دیگری رو ورکر میزاریم

<h2> task.js </h2>

خب تو این فایل تغییرات زیادی نداریم و مثل برنامه ی ارسال کننده ی پیام هستش منتها چند نکته داره که بعد از کد بهشون میپردازیم

</div>

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
    
    const queue = "task"
    const msg = process.argv.slice(2).join(' ') || "Hello World!";
    
    channel.assertQueue(queue, {durable: true});
    
    channel.sendToQueue(queue, Buffer.from(msg), { persistent: true });
    console.log(" [x] Sent %s", msg);
  });
  setTimeout(function() {
        connection.close();
        process.exit(0);
    }, 500);
})

```
<div dir='rtl'>

اولین تغییر در این خط هستش

</div>

```javascript
const msg = process.argv.slice(2).join(' ') || "Hello World!";
```
<div dir='rtl'>

ما در این خط گفتیم که ورودی رو از خود کاربر بگیره به این صورت که اگه موقع اجرا کردن برنامه کلمه ای جلوی اسم برنامه تایپ کنه اون به عنوان پیام ارسال میشه و در غیر این صورت ما سلام دنیا رو ارسال میکنیم

خط بعدی که تغییر کرده و نیازه در موردش صحبت کنیم این تیکه از کد هستش

</div>

```javascript
channel.assertQueue(queue, {durable: true});
    
channel.sendToQueue(queue, Buffer.from(msg), { persistent: true });
```
<div dir='rtl'>

قبلا راجب دوریبل صحبت کردیم و گفتیم که برای این به کار میره که وقتی سرور رابیت داون شد یا برنامه کرش کرد صف رو ذخیره کنه و صف از بین نره همین طور هم که مشاهده میکنید تو این قسمت کد بر خلاف برنامه ی قبلی ما این قسمت رو تورو کردیم تا صفمون ذخیره بشه 

یه اپشنی که به متد ارسال کننده ی پیام تو صف اضافه شد پرسیستنت هستش که این هم وظیفه ی ذخیره ی پیام ها وقتی که سرور داون شد یا کرش کرد رو داره

خب فرقش با دوریبل اینه که دوریبل صف رو ذخیره میکنه و پرسیستنت خود پیام ها رو ذخیره میکنه به این صورت که وقتی سرور داون شد پیام ها داخل صف میمونن تا وقتی سرور بالا بیاد و پیام ها دونه دونه به ورکر ها داده بشن

خب این قسمت نکته ی خاص دیگه ای نداره و قبلا بقیه مواردشو توضیح دادیم

<h2> worker.js </h2>

در این فایل هم ما قسمت های مشابه زیادی داریم که قبلا توضیح دادیم و کدش رو میتونید مشاهده کنید

</div>

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
    
    const queue = "task"
    const msg = process.argv.slice(2).join(' ') || "Hello World!";
    
    channel.assertQueue(queue, {durable: true});

  });
})

```
<div dir='rtl'>

حالا میریم سراغ مباحث جدید 

</div>

```javascript
channel.prefetch(1);
```
<div dir='rtl'>

این اولیثن مبحثی هستش که میخوایم باهم بررسیش کنیم

پریفیچ یک , وظیفه ی ارسال کردن یک پیام از صف به ورکر رو داره 
خب این یعنی چی

بزارید یه تیکه ی دیگه ی کد رو هم ببینیم باهم تا بتونیم دقیق بفهمیم که پریفیچ چه کاری انجام میده و چرا ما ازش استفاده میکنیم

</div>

```javascript
console.log(" [*] Waiting for messages in %s. To exit press CTRL+C", queue);
    channel.consume(queue, function(msg) {
      console.log(" [x] Received %s", msg.content.toString());
    }, {
      noAck: false
    });
```

<div dir='rtl'>

ما قبلا این تیکه کد رو تو درس قبلی دیده بودیم ولی باز یه توضیح مختصری راجبش میدیم

متد کانسیوم اسم صف و یک فانکشین و اپشنی رو میگرفت و ما با این متد میتونستیم به پیامی که برامون ارسال شده دسترسی داشته باشیم

اگه دقت کنید تنها فرقی که این کد با کد فایل درس قبل داره تو قسمت اکت اون هست

در این قسمت اکت مساویه فالس هست و اگه به یاد بیارید گفته بودیم وظیفه ی اکت این هست که وقتی پیام رو به یک ورکر یا دریافت کننده ی پیام میده پیام رو از داخل صف برمیداره 

ولی خب اینجا یه سوالی برای ما پیش میاد

اگه ما سه تا پیام داشته باشیم و دوتا ورکر و پیام اولی رو به ورکر اولی و پیام دومی رو به ورکر دومی بدیم در این حالت اگه پرفیچ نداشته باشیم و اکت مساوی تورو باشه با اینکه پیام قبلی هنوز انجام نشده باز یک پیام جدید به ورکر اولی داده میشه

پس ما تو این حالت اکت رو فالس میکنیم و خودمون به صورت دستی تو فانکشنمون میگیم که کارمون تموم شد و میتونی پیام رو از تو صف برداری و پریفیچمون هم یک هست و هر بار یک پیام به ورکر ما میده 

یکی دیگه از مزایای این کار این هست که وقتی پیامی به ورکر داده بشه پیام تو صف میمونه و اگر ورکر ما داون بشه یا به هر دلیلی کرش کنه اون پیام تو صف هست و وقتی ورکر های بعدی سرشون خلوت بشه و کارشون رو انجام بدن پیام رو میگیرن و انجام میدن یا حتی اگه ورکر دیگه ای وجود نداشت و فقط یک ورکر داشتیم پیام داخل صف متنظر میمونه تا ورکر دوباره بالا بیاد تا دوباره بهش پیام ارسال بشه

پس ما این تیکه کد رو که بصورت دستی اکت میفرستیم بعد از اتمام کارمون رو به کدمون اصافه میکنیم

</div>

```javascript
console.log(" [*] Waiting for messages in %s. To exit press CTRL+C", queue);
    channel.consume(queue, function(msg) {
      console.log(" [x] Received %s", msg.content.toString());
      channel.ack(msg);
    }, {
      noAck: false
    });
```
<div dir='rtl'>

بیاین یه کار دیگه هم انجام بدیم تا بتونیم درست این کد هایی که زدیم رو درک کنیم و مفهوم صف و ورکر ها رو متوجه بشیم

بیاین تو این فانکشینی که تو کانسیوم تعریف کردیم کدی بنویسم که وقتی کسی که میخواد پیام بفرسته جلوی فایل کنار پیام هر چند تا نقطه گذاشت به ازای هر نقطه ۱ ثانیه ورکر ما مشغول به کار بشه تا بتونیم درک کنیم که کار در ورکر طول کشیده و نوبت گردشی به چه صورت انجام میشه
</div>

```javascript
channel.consume(queue, function(msg) {
      const secs = msg.content.toString().split('.').length - 1;

      console.log(" [x] Received %s", msg.content.toString());
      setTimeout(function() {
        console.log(" [x] Done");
        channel.ack(msg);
      }, secs * 1000);
    }, {
      noAck: false
    });
```

<div dir='rtl'>

خب اگه دقت کنید ما یک ثانیه تعریف کردیم که به تعداد نقطه هایی که کاربر جلوی پیام خود اضافه میکنه همون تعداد هم تو فانکشین ما اسلیپ بشه تا بتونیم متوجه بشیم که داخل ورکر ها چی میگذره

حالا وقتشه برنامه هامون رو ران کنیم

من برای مثال دستور هر دو برنامه رو براتون مینویسم تا متوجه بشید که چجوری باید رانش کنید

اول دوتا ترمینال باز کنید و تو هر دوی اونها این دستور را وارد کنید تا دوتا ورکر داشته باشیم

</div>

```
node worker.js
``` 

<div dir='rtl'>


بعد از ران کردن و اماده کردن دوتا ورکر خودمون وقتشه که یه تسک رو بزاریم تو صف تا ورکرهامون انجامش بدن

</div>

```
node task.js thisIsATask..........
```

<div dir='rtl'>

خب همین طور که میبینید من جلوی پیامم چندین نقطه گذاشتم که به این معنی هست وقتی که پیام به ورکر رسید طبق این کد زیر به همون تعداد نقطه ورکر اسلیپ بشه تا ما بتونیم چندین پیام دیگه داخل صف قرار بدیم تا بتونیم الگریتم نوبت گردشی رو بین ورکر ها ببینیم و ببینیم که وقتی که کار یک تسک انجم شد تسک بعد چگونه بهش داده میشه یا اگه یک ورکر در حال انجام یک تسک داون شد تسک به ورکر بعدی ارسال میشه یا نه


</div>

```javascript
const secs = msg.content.toString().split('.').length - 1;
```

<div dir='rtl'>

خب وقتی که دستور بالا رو با همون تعداد نقطه ارسال کردید میتونید باز با تعداد نقطه های متفاوت و متن متفاوت پیام داخل صف قرار بدید 

بعد از گذاشتن چندین پیام داخل صف منتظر بمونید تا تسک ها دونه دونه توسط ورکر ها انجام بشن 

بعد تست های مد نظرتون رو انجام بدید به این شکل که وقتی ورکر در حال کار کردن است بریکش(۳) کنید تا ببینید تسکی که در دست خود ورکر هست به ورکر بعدی داده میشه یا نه

من کل کد ورکر رو هم در اینجا قرار میدم تا بتونید به راحتی کپیش کنید و تستش کنید


</div>

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
    
    const queue = "task"
    const msg = process.argv.slice(2).join(' ') || "Hello World!";
    
    channel.assertQueue(queue, {durable: true});
    
    channel.prefetch(1);
    console.log(" [*] Waiting for messages in %s. To exit press CTRL+C", queue);
    
    channel.consume(queue, function(msg) {
      const secs = msg.content.toString().split('.').length - 1;

      console.log(" [x] Received %s", msg.content.toString());
      setTimeout(function() {
        console.log(" [x] Done");
        channel.ack(msg);
      }, secs * 1000);
    }, {
      noAck: false
    });
  });
})

```

---
(۱) worker

(۲) round-robin

(۳) break => Ctrl+c
