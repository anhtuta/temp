# 1. SOLID principle

- **S**ingle responsibility principle: Một class chỉ nên giữ 1 **trách nhiệm duy nhất**. Ngoài ra, chỉ sửa class đó với 1 lý do duy nhất

VD: thay vì 1 class chứa các method sau:

```java
public class DatabaseHelper {

  public Connection openConnection() {};

  public void saveUser(User user) {};

  public List<Product> getProducts() {};

  public void closeConnection() {};
}
```

Ta nên tách thành 3 class nhỏ hơn như sau:

```java
public class DatabaseHelper {
  public Connection openConnection() {};
  public void closeConnection() {};
}

public class UserRepository {
  public void saveUser(User user) {};
}

public class ProductRepository {
  public List<Product> getProducts() {};
}
```

- **O**pen/closed principle: Có thể thoải mái **mở rộng** 1 class, nhưng **không được sửa đổi** bên trong class đó (open for extension, but closed for modification):

Nói cách khác: không được thay đổi hiện trạng của các lớp có sẵn, nếu muốn thêm tính năng mới, thì hãy mở rộng bằng cách kế thừa để xây dựng class mới

VD: class sau quản lý connection tới database, ban đầu hệ thống dùng 2 CSDL là MySQL và SQL Server:

```java
class DatabaseConnection {
  public void connect(String db) {
    if(db == "SQLServer") {
       //connect with SQLServer
    } else if(db == "MySQL") {
      //connect with MySQL
    }
  }
}
```

Nếu như sau này cần thêm mongodb nữa thì phải sửa lại method `connect` đó, tức là phải sửa lại code cũ => ảnh hưởng tới code cũ đang chạy ngon lành

Giải pháp: thiết kế lại class DatabaseConnection dùng abstract:

```java
abstract class DatabaseConnection {
  public abstract void connect();  // ko cần param db nữa
}
class MySQLConnection extends DatabaseConnection {
  public void connect() {
    // connect with MySQL
  }
}
class SQLServerConnection extends DatabaseConnection {
  public void connect() {
    // connect with SQLServer
  }
}
public static void main(String[] args) {
  DatabaseConnection dc = new MySQLConnection();
  dc.connect();
}
```

Nếu sau này thêm mongodb thì có thể tạo 1 class MongoConnection kế thừa DatabaseConnection, tức là ko phải sửa lại bất cứ phần code nào đã có sẵn:

```java
class MongoConnection extends DatabaseConnection {
  public void connect() {
    // connect with Mongodb
  }
}
```

- **L**iskov substitution principle: Trong một chương trình, các object của **class con có thể thay thế class cha** mà không làm thay đổi tính đúng đắn của chương trình (substitution: sự thay thế)

VD1: 1 method có 1 param kiểu `List`: `public void doSomething(List list)`, thì khi gọi và truyền 1 object ArrayList nó vẫn hoạt động được: `doSomething(new ArrayList())`

VD2: đoạn code sau sẽ vi phạm nguyên tắc này (`class Dog` là con nhưng ko thể thay thế cha là `class Animal`)

```java
public abstract class Animal {
    protected String name;

    abstract void eat();
    abstract void fly();
}

public class Bird extends Animal {

    @Override
    void eat() {
        System.out.println("Bird " + name + " is eating");
    }

    @Override
    void fly() {
        System.out.println("Bird " + name + " is flying");
    }

}

public class Dog extends Animal {

    @Override
    void eat() {
        System.out.println("Dog " + name + " is eating");
    }

    @Override
    void fly() {
        throw new UnsupportedOperationException("Dog cannot fly!!!");
    }

}
public static void main(String[] args) {
    Animal animal1 = new Bird("Phụng hoàng bất tử");
    animal1.eat();
    animal1.fly();  // Bird Phụng hoàng bất tử is flying

    Animal animal2 = new Dog("Sói xám");
    animal2.fly();  // Exception in thread "main" java.lang.UnsupportedOperationException: Dog cannot fly!!!
}
```

=> Rõ ràng class Dog đã vi phạm nguyên lý thay thế Liskov, bởi vì 1 object Dog ko thể thay thế object Animal nếu object Dog gọi method fly()

Cách giải quyết: tách method fly ra 1 interface riêng, động vật nào biết bay mới cho phép implement interface đó:

```java
public abstract class Animal {
    protected String name;

    abstract void eat();
}
public interface Flyable {
    void fly();
}

public class Bird extends Animal implements Flyable {

    @Override
    void eat() {
        System.out.println("Bird " + name + " is eating");
    }

    @Override
    public void fly() {
        System.out.println("Bird " + name + " is flying");
    }

}

public class Dog extends Animal {

    @Override
    void eat() {
        System.out.println("Dog " + name + " is eating");
    }

}
```

- **I**nterface segregation principle: Thay vì dùng 1 interface lớn, ta nên **tách thành nhiều interface nhỏ**, với nhiều mục đích cụ thể (segregation: sự tách biệt)

VD1: Trong ví dụ Animal ở trên, ta đã tách class Animal thành 1 class và 1 interface, vì ko phải Animal nào cũng có thể fly()

VD2:

```java
interface Phone {
    public function call();
    public function sms();
    public function takePicture();
}
```

Hiển nhiên điện thoại 1280 ko thể chụp ảnh được, nên ta nên tách interface đó thành 3 interface nhỏ hơn: `Callable, Smsable, TakePictureable`

- **D**ependency inversion principle (inversion: sự đảo ngược):
  - Các module cấp cao **không** nên **phụ thuộc vào các modules cấp thấp**. Cả 2 nên phụ thuộc vào abstraction
  - Các class giao tiếp với nhau thông qua _interface_, không phải thông qua implementation
  - Những thành phần trừu tượng không nên phụ thuộc vào các thành phần mang tính cụ thể mà nên ngược lại
  - Những cái cụ thể dù khác nhau thế nào đi nữa đều tuân theo các quy tắc chung mà cái trừu tượng đã định ra. Việc phụ thuộc vào cái trừu tượng sẽ giúp chương trình linh động và thích ứng tốt với các sự thay đổi diễn ra liên tục.
  - Là nguyên tắc khó hiểu nhất trong SOLID

Trong lập trình truyền thống, các _module cấp cao_ sẽ **gọi** đến các _module cấp thấp_  
=> _Module cấp cao_ sẽ **phụ thuộc** và _module cấp thấp_  
=> Tạo ra các dependency  
=> Khi các module cấp thấp thay đổi => các module cấp cao phải phải thay đổi theo  
=> Làm tăng tính phức tạp của code và khó để bảo trì

Nếu áp dụng Dependency Inversion: các module sẽ giao tiếp với nhau thông qua interface  
=> Cả module cấp cao và cấp thấp sẽ không phụ thuộc lẫn nhau mà phụ thuộc vào interface  
=> Có thể dễ dàng thay thế các implementation của các module cấp thấp miễn là chúng đều triển khai một interface

# 2. JWT

## 2.1. Lưu JWT ở đâu

### Local Storage, session storage

- Quá nguy hiểm, vì có thể được truy cập từ bất kỳ script nào => Dễ bị dính XSS
- **Không** có thời gian expire => ko tốt

### Cookie

Dù cookie vẫn có thể truy cập từ script trong cùng domain bằng cách gọi `document.cookie`. Nhưng nó vẫn an toàn hơn localStorage.

- Có expiration
- Có flag `httpOnly`: giúp ngăn chặn cookie truy cập bởi client side (KHÔNG thể dùng `document.cookie` để đọc được nữa)

Nhưng dùng cookie sẽ bị dính CSRF. Ta cần làm 3 bước sau để phòng tránh:

- Set flag `SameSite=Strict`: flag này chỉ cho phép gửi cookie đi trên cùng 1 domain đã set mà thôi. Nhược điểm: không bật được tính năng cross-site như social login các thứ cần kết nối tới 1 service 3rd party.
- Set flag `Secure=true`: flag này chỉ cho phép gửi cookie đi qua kết nối HTTPS
- Phòng tránh CSRF phía server: như trong Java spring security, thì nó sẽ thêm 1 field input hidden có name là `_csrf` như sau:
  ```html
  <input
    type="hidden"
    name="_csrf"
    value="4bfd1575-3ad1-4d21-96c7-4ef2d9f86721"
  />
  ```
  Chỉ có server mới generate và verify cái value đó được

Client gửi token đi thế nào, khi đã set `httpOnly=true`? Hãy dùng header này phía client: `Access-Control-Allow-Credentials: true`. Khi header này được set true, trình duyệt sẽ tự động gửi bất kỳ cookie nào có thuộc tính `HttpOnly` mà có domain là domain đang gửi request tới.

Với axios, chúng ta có thể enable bằng cách set withCredentials: true:

```js
axios.get("https://api.domainA.com", { withCredentials: true });
```

Ref: https://viblo.asia/p/authentication-voi-jwt-luu-token-o-dau-la-bao-mat-nhat-aNj4vz2v46r

## 2.2. Verify jwt như thế nào

Jwt sẽ có 3 phần sau (header và payload đều đc encode base64):

- Header: lưu thuật toán hash, ex: HS256, RS256...
- Payload: chứa claims và identifying information (thường là user info)
- Signature: đc hash như sau: `HMACSHA256(header.payload, your-256-bit-secret)` (hash chuỗi `header.payload` bằng key `your-256-bit-secret`)

Phía server sẽ nhận đc jwt sẽ verify như sau:

- Nếu dùng thuật toán hash đối xứng như HS256 (symmetric):
  - Server sẽ dùng chính privateKey dùng lúc gen jwt để verify
  - Server sẽ hash lại signature: lấy header + payload từ jwt và privateKey. Nếu signature này trùng với signature của jwt thì jwt đó hợp lệ
- Nếu dùng thuật toán hash bất đối xứng như RSA256 (asymmetric):
  - Server uses a **private key to sign** a JWT
  - Server uses a **public key to verify** that signature

Ref:

- https://www.freecodecamp.org/news/how-to-sign-and-validate-json-web-tokens/
- https://auth0.com/blog/rs256-vs-hs256-whats-the-difference/

# 3. Compare RabbitMQ vs Apache Kafka

Queue vs Topic

- RabbitMQ: queue
- Kafka: topic

Using Kafka for a task queue is a bad idea. Use RabbitMQ instead, it does it much better and more elegantly. Why?

- Kafka is not allowing to consume a single partition by many consumers. Do đó nếu như 1 partition bị gửi rất nhiều task vào thì consumer đọc message từ partition đó sẽ rất busy, nhưng các consumer khác thì lại rảnh
- Kafka topic là 1 message-log chứ ko phải là queue: Thứ tự mà consumer tiêu thụ task trong topic sẽ khác với thứ tự mà producer nhét vào, nên nếu các task cần thực thi theo đúng thứ tự nhét vào, thì ko nên dùng kafka

Protocol:

- RabbitMQ **supports several standardized protocols** such as AMQP, MQTT, STOMP...
- Kafka uses a **custom protocol**, on top of TCP/IP for communication between applications and the cluster. **Kafka can’t simply be removed and replaced**, since its the only software implementing this protocol.

Routing:

- RabbitMQ: **complex ways of routing**, such as: direct, fanout, topic, header
- Kafka **does not support routing**; Kafka topics are divided into partitions which contain messages in an unchangeable sequence (Kafka topic đc chia thành các partition chứa các message theo 1 trình tự ko đổi)
  - You can create dynamic routing yourself with help of Kafka streams, where you dynamically route events to topics, but it’s not a default feature

Persistence

- RabbitMQ: Persist messages until they are dropped on the acknowledgement of receipt (chỉ persist message trc khi nó dc tiêu thụ hoặc trc khi nhận dc ack). Messages are stored until a receiving application connects and receives a message off the queue. The client can either ack (acknowledge) the message when it receives it or when the client has completely processed the message. In either situation, once the message is acked, it’s removed from the queue.
- Kafka: Persists messages with an option to delete after **a retention period** (In other words: message queue in Kafka is persistent. The data sent is stored **until a specified retention period has passed**, either a period of time or a size limit)

Replay

- RabbitMQ: No
- Kafka: Yes. But ensure that you are using it in the correct way and for the correct reason
  - Should NOT replaying an event multiple times that should just happen a single time.
  - Ex1: Save a customer order multiple times => should NOT
  - Ex2: You have a bug in the consumer that requires deploying a newer version, and you need to re-processing some or all of the messages => Good choice (Nếu consumer bị lỗi cần deploy lại, thì việc replay sẽ giúp đọc lại các message trc đó đã đọc lỗi)

Message Priority

- RabbitMQ: Support priority queues
  - The priority of each message can be set when it is published.
  - Depending on the priority of the message it is placed in the appropriate priority queue
- Kafka: No
  - A message cannot be sent with a priority level, nor be delivered in priority order.
  - All messages in Kafka are **stored and delivered in the order in which they are received** regardless of how busy the consumer side is.

How to work with the queues?

- RabbitMQ's queues are fastest when they're empty
- Kafka is designed for holding and distributing large volumes of messages. Kafka retains large amounts of data with very little overhead.
- RabbitMQ's **lazy queues**: the messages are automatically **stored to disk**, thereby minimizing the RAM usage, but extending the throughput time.
  - In our experience, lazy queues create a more stable cluster with better predictive performance.
  - If you are **sending a lot of messages** at once (e.g. processing **batch** jobs), or if you think that your **consumers will not consistently keep up with the speed of the publishers**, we recommend that you enable lazy queues.
- (Lazy queue của RabbitMQ giống topic của Kafka ở chỗ nó cũng lưu message trên disk, đỡ tốn RAM. Nếu send quá nhiều message, hoặc tốc độ consumer ko theo kịp producer, thì nên dùng lazy queue)

CONSUMER SCALING

- RabbitMQ: simply adding and removing consumers
  - If you publish quicker then you can consume, your queue will start to grow and might end up with millions of messages, finally causing RabbitMQ to run out of memory.
  - In this case, you can scale the number of consumers that are handling (consuming) your messages
  - Each queue in RabbitMQ can have many consumers, and these consumers can all “compete” to consume messages from the queue
  - The message processing is spread across all active consumers => scaling up and down in RabbitMQ can be done by simply adding and removing consumers.
- Kafka: the way to distribute consumers is by using topic partitions
  - Each consumer in a group is dedicated to one or more partitions
  - 1 partition can be consumed by only 1 consumer
  - Do đó: muốn tạo nhiều consumer => phải có nhiều partition => send message bởi nhiều key riêng biệt: send each partition different set of messages by business key, ex: by user id, location...

SCALING BROKER

- RabbitMQ is mostly designed for vertical scaling (adding more power) in mind
  - Horizontal scaling is possible in RabbitMQ, but that means that you must set up clustering between your nodes, which will probably slow down your setup
- Kafka is built from the ground up with horizontal scaling (adding more machines) in mind
  - You can scale by adding more nodes to the cluster or by adding more partitions to topics

Monitoring built-in

- RabbitMQ: Yes
  - It has a user-friendly interface that lets you monitor and handle your RabbitMQ server from a web browser. Among other things, queues, connections, channels, exchanges, users and user permissions can be handled (created, deleted, and listed) in the browser, and you can monitor message rates and send/receive messages manually.
- Kafka: No
  - Must use other open-source tools

PUSH or PULL

- RabbitMQ: Messages are **pushed** from queue to the consumer.
  - Consumers can also pull messages from RabbitMQ, but it’s not recommended.
- Kafka: uses a **pull** model
  - Consumers request batches of messages from a given offset

Complexity

- Personally, I thought it was easier to get started with RabbitMQ and found it easy to work with. And as a customer of ours said:
  > "We didn't spend any time learning RabbitMQ and it worked for years. It definitely reduced tons of operational cost during DoorDash's hyper-growth." Zhaobang Liu, Doordash
- In my opinion, the architecture in Kafka brings more complexity, as it does include more concepts such as topics/partitions/message offsets, etc. from the very beginning. You have to be familiar with consumer groups and how to handle offsets.

Use cases

- RabbitMQ: if you want a simple/traditional pub-sub message broker for:
  - LONG-RUNNING TASKS: Message queues enable **asynchronous** processing, meaning that they allow you to put a message in a queue without processing it immediately. Ex:
    - Images Scaling
    - Sending large/many emails
    - Search engine indexing
    - File scanning
    - Video encoding
    - Delivering notifications
    - PDF processing
    - Calculations
    - Simple flow: a service send a request to a queue, then another service listen to it and read message and perform processing task
  - MIDDLEMAN IN A MICROSERVICE ARCHITECTURES: it serves as a means of communicating between applications, avoiding bottlenecks passing messages
- Kafka: if you want a framework for storing, reading (re-reading), and analyzing streaming data
  - DATA ANALYSIS: TRACKING, INGESTION, LOGGING, SECURITY: large amounts of data need to be collected, stored, and handled
  - REAL-TIME PROCESSING: it could be used in systems handling many producers in real-time with a small number of consumers, ex: financial IT systems monitoring stock data

Ref:

- https://www.cloudamqp.com/blog/when-to-use-rabbitmq-or-apache-kafka.html
- https://www.cloudamqp.com/blog/rabbitmq-use-cases-explaining-message-queues-and-when-to-use-them.html
- https://stackoverflow.com/questions/36206204/is-apache-kafka-appropriate-for-use-as-an-unordered-task-queue
- https://stackoverflow.com/questions/35561110/can-multiple-kafka-consumers-read-same-message-from-the-partition
