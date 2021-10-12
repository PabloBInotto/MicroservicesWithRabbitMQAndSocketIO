# MicroservicesWithRabbitMQAndSocketIO
In this application, we will try to informed customers who buy the stock papers when the price changed. 
We will use NodeJs for the backend application. For improving the performance, we will use RabbitMQ, 
and we will write microservice as a consumer. We will send updated stock data by using SocketIO to the clients. 
For front-end application, we will use Angular 9

Today we will talk about real-time information used in all areas of social life. Speed ​​is the most important thing in the new technology century. In this article, we will try to informed customers who buy the stock papers when the price changed. It should be as fast and accurate as possible.
You can listen to the radio to get information about the exchange news once in a day like above the yellow taxi radio. But if you want to learn something in a real-time, you have to do more then this. You have to use technology. We will use “Socket.IO” in this example.
“There is more to life than simply increasing its speed.”
― Mahatma Gandhi
But speed is not enough. Performance should also be. What is the performance of programming? Completing the job within a required time period. The user experiences are no interrupted or slowdown. How is this accomplished?
“What Rome did is to divide a great system into several microsystems, thus diluting the entropy that can affect the great system by reducing it.”
Divide and conquer…

This is a bronze statue of the first Roman emperor Caesar Augustus

How could we improve performance? If we separate some long processes, we keep customers waiting less while working. We will use Microservices for performance. Informing customers about the exchange of stock data is a very intense process. We will add to a Queue all inserted and updated stock data. We will use “RabbitMQ” for this application. We will write consumer microservice and get data from the queue and finally notify all customers with Socket.IO.
We will use NodeJs for the Backend.
Let’s create the first step of this new project:
Setup Before Starting Backend Application For Macbook :
Firstly install NodeJs. We will use Socket.IO for real-time information service.
Install Visual Studio Code: We will use VSCode for IDE. I love it. It is fast and straightforward.
Install HomeBrew for installing RabbitMQ.
“brew install rabbitmq”: Install Rabbitmq. I will use RabbitMQ to queue the relevant data.
“rabbitmq-server”: Run RabbitMQ.

If everything is ok, when you type “http://localhost:15672” from the browser and enter Username:” guest,” Password:” guest,” you have to see a screen like below the picture.

This is enough for the First Step :) Microservice Architecture is built on the simple working principle. But in fact, it is a combination of lots of complex pieces.
“Simplicity is the ultimate sophistication.”
-Leonardo Da Vinci
Now let’s create the first NodeJs Backend Service:
Lorem Ipsum Stock Data-DB
stockData.js: For not to distribute our focus, I will not get data from any database, like MsSql, Oracle, or MongoDB. So let’s create our static Lorem Ipsum Stock Data list.

RabbitMQ

“npm install amqplib — save” : We have to install amqplib library for using RabbitMQ on NodeJs.
rabbitMQ.js: This is the queue in which the changed data will push to the clients.

1-) This function gets the channel name and updated stock data as a parameter..

rabbitMQ.js(1)
2-)This is the local path of RabbitMQ.

rabbitMQ.js(2)
3-)We are creating a channel on connected RabbitMQ.

rabbitMQ.js(3)
4-)Checks for “queueName (updateStock)” queue. If it doesn’t exist, then it creates one.

rabbitMQ.js(4)
5-)Put the stock data onto the “queueName (updateStock)” queue.

rabbitMQ.js(5)
NodeJS Service

service.js : This is our backend service. We send and update all stock data here. And we put modified stock data in a “updateStock” channel on RabbitMQ.
“npm install express — save” : We will use the Express library on NodeJs for all HTTP requests.
“npm install body-parser” : Parse incoming request bodies in a middleware before your handlers, available under the req.body property.
“npm install — save cors” :It is used for allowing Cross-Origin. A client can get or post request from a browser to NodeJs service.

1-) This is the bodyParser library. It is available to get the updated stockData by req.body

service.js(1)
2-) This function is used for getting static Stock Data from the client. And all the data are sorted by the name.”localeCompare” is a javascript function. It is used for comparing two strings and returns a number indicating whether the string comes before, after, or is equal as the compare string in sort order.

service.js(2)
3-) When the administrator updates the properties of a stock, updated stock data from the client is posted to this “updatestockData” service as a parameter. “req.body,” is used for getting this parameter from the request. And finally, current stock data is updated with “findAndUpdateStock()” function. We will talk about it in the next item.

service.js(3)
4-) This function is used for finding and removing old data by the name from the static data list. Finally, updated data is added to the same list.

service.js(4)
5-) To informing the customers, updated data is put on the “updateStock” channel in RabbitMQ. Don’t forget to convert data to string with the “JSON.stringify()” function before putting it on the channel.

service.js(5)
6-) This NodeJs web service is published from 9480 port.

service.js(6)
“Microservices are a kind of asynchronous mechanism.
Nobody waits for the process to the end, and they always work at the behand of the scene.”
— Bora Kaşmer
Consumer

consumer.js: I compare the Consumer to a worker who works forever :) Consumer is an application or an instance that consumes messages. The same application can also publish messages and thus be a publisher at the same time. In this application, we will consume stock data from the “updateStock” channel, and at first, we will just press the stock’s name and value on to the screen. Later, we will notify this updated data to all clients.

1-)The ”io” is our custom Socket module. It is used for notifying the customers. “ampq” is a RabbitMQ library. It is used for pulling the updated data from the queue.

consumer.js(1)
2-) This function “amqp.connect()” is used for connect to local RabbitMQ serve.

consumer.js(2)
3-) This function is used for creating the “updateStock” channel, which we will take from stock data.

consumer.js(3)
4-) We declare a queue for consuming the data. “durable” means RabbitMQ saves the data on the memory as a default. But if we set “durable: true,” queues are persisted to disk. So If the server is taken down and then brought back up, the durable queue will be re-declared during server startup, however, only persistent messages will be recovered. Other On memory(durable: false) Queues will be lost.

consumer.js(4)
5-) This “channel.consume ()” function is to listen to the “updateStock” channel, get the updated stock data from the queue and send it to all customers in real-time using the Socket.IO service that we have not written yet.
noAck: true means, when the data is consumed, it will remove from the queue.

consumer.js(5)
io.socket.emit(“updatedStock”, stock) :
6-) This upper function is used for triggering the “updateStock” client-side function of all connected clients. And it sends them the updated stock data on Real-Time.
Socket.IO

socket.js: This module export a Socket Server. We will use it on Consumer. When the data is pulled from the queue, we will push it to all clients by using this Socket Server.
"npm install express socket.io --save": We need the Socket.IO library to inform the updated socket data to all clients.

1-) Io Socket Server needs a server, and the server needs an express library, so we declare all of them like seen belove the picture.

socket.js(1)
2-) When all the clients connect to Socket-IO, this “io-on(‘connecton’)” function will be triggered.

socket.js(2)
3-)When one of the clients is logout, this “socket, on (“disconnect”)” function will be triggered.

socket.js(3)
4-) Socket-IO and Consumer are published from 1923 port.

socket.js(4)

Image Source
Front-end Angular
Backend is finished. Let’s create the Front-End application. We will use Angular 9 for this application.
"npm i ngx-socket-io" : We need the ngx-socket-io library to get updated stock data and give real-time information to all clients.
"npm install bootstrap --save" : I use bootstrap for the css library. 
Note: Add "style.css": @import "~bootstrap/dist/css/bootstrap.css";

Create Angular StockUI application with this command!
Model
stock.ts: Create stock model, which is monitoring by the clients.
export class Stock {
    name: string;
    value: number;
    change: number;
    percentage: number;
    image:string;
}
Service
stockService.ts: It is used for getting and updating stock data from NodeJs. And It listens to the “updateStock” event to notify the client with real-time stock data.

1-) HttpClient and Socket libraries are injected into the “stockService” on the Constructor. It listens for the “updatestock” event and assigns the updated stock data to the “updateStock” variable when it comes from Socket.IO. When consumers received the updated stock data from the rabbitMQ, it will emit this to the client-side “updateStock” event.

stockService.ts(1)
2-)getStocks() function is used for getting all stock data from the NodeJs service. Return type Observer List of Stock. And if an error occurs, It tries to get the data one more time. More then 2 errors it calls “errorHandel()” function.

stockService.ts(2)
3-) It is used for error handling. You can send mail to the system desk or add as a document into an index of elastic-search.

stockService.ts(3)
AppComponent
app.component.ts:

1-)AppComponent is inherited from “AfterViewInit” because after the page is loaded, I want to get all stock data. The “ngAfterViewInit()” method provides this to me. It is like “Windows Form.OnLoad” method. And it comes from the AfterViewInit interface.

app.component.ts(1):
2-)This method is used for getting all stock data from the NodeJs service. It is an observer async method. After we got all data, we assigned all of them to the “stockList” Stock[] array.
The “image” column is only our view-model property. It is used for showing a green or red arrow icon on the page, which depends on the stock change value. There is no similar property in NodeJs service for this column.

app.component.ts(2):
3-)After the page is loaded, all stock data takes from the NodeJs service.
We declared the “updateStock” service, which is listening to the “updateStock” event. When update data come from the NodeJs service, we set the image property to depend on the change value of this stock data. And after all, we will remove the old one from our stockList and add an updated newest one to the same list.

app.component.ts(3):
Finally, I didn’t write any Admin page or UpdateService on Angular for updating stock data. Instead of this, I used “Postman.” I post stock data as above picture to “updateStockData()” NodeJs service.

Postman

Stock Monitoring Screen
app.component.html: I will iterate over the stocklist and render Html table row for every stock item on the screen.
<tbody>
   <tr *ngFor="let stock of stockList">
      <td>{{stock.name}}</td>
      <td>{{stock.value}}</td>
      <td>{{stock.change}}</td>
      <td>{{stock.percentage}}</td>
      <td><img src="../assets/images/{{stock.image}}.png" width="50px"></td>
   </tr>
</tbody>
AppModule
app.modules.ts: All angular service, component, socket and module definitions are in this module.

1-) For using SocketIo on Angular, we need the “ngx-socket-io” library. And we have to define the Backend SocketIO HTTP server address as a config. In the end, we will use this config as a forRoot’s parameter during import this SocketIoModule.

app.modules(1)
2-)Forms module is used for model and form element-binding. HttpClient module is used for http.post and http.get request. SocketIoModule is used for Real-Time Communication between SocketIO and Angular.

app.modules(2)


