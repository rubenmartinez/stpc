# btc-market-exchange-viewer

Just a practice project to experiment with multithreading Java 8 features: A webapp that connects to a crypto exchange REST API and shows the current state of the market.

## Build/Run application

#### Docker

    docker run -p8080:8080 rubenmartinez/stpc

Then point your browser to: [http://127.0.0.1:8080](http://127.0.0.1:8080) when the log says server is ready. (Note: Microsoft InternetExplorer/Edge are not supported)

#### Maven Build

Just execute in the base directory:

    mvn clean install

Then the server app can be executed with:

    java -jar stpc-app-1.0.0.jar

Then point your browser to: [http://127.0.0.1:8080](http://127.0.0.1:8080) as in the Docker run.

##### Notes on Unit Testing

JUnit 5 is used for Unit Testing so mininum Apache Maven version required is 3.3.9. You can always use `-DskipTests=true` to skip tests compilation and execution. This also speeds up building times considerably.

The effort and focus has been put in testing the most <em>critical</em> logic under all possible conditions, more than trying to get a high percentage of coverage just for the looks, trivial classes have been left as a lower priority while complex classes have much more test coverage (and even with several parametrized conditions that really don't increase the test coverage much).

TODO/Note: tests might fail under some circumstances as they have some assumptions on async code durations using parallel threads, even if the "timeouts" should be high enough, on slow machines a timeout for an async code could trigger so the test is assumed to fail, even if the code is working correctly.


#### IDE notes

##### Unit Testing

As JUnit 5, old versions of Eclipse or IntelliJ IDEA won't work for testing. At least **Eclipse Photon**, or **IntelliJ IDEA 2016.2** are required.

Actually Eclipse Photon didn't support well JUnit5 in some circumstances. For this project I had to replace $ECLIPSE_HOME/plugins/org.hamcrest.core_1.3.0.v20180420-1519.jar with hamcrest-core-1.3.jar from Maven Repository (See [StackOverflow thread](https://stackoverflow.com/questions/9651784/hamcrest-tests-always-fail))

##### Lombok

Originally, the project made extensive use of [Lombok](https://projectlombok.org/) to improve final readability and development speed.

The caveat is that the IDE needs to have Lombok installed so all classes can be compiled. Even if installation is really easy, in order to not make it mandatory to install Lombok for the reader, the project have been *de-lomboked*, that why many classes have a comment `Generated by delombok at XXX`

---

## Modules

The project is divided into 3 modules, so the application and strategies are decoupled from the actual exchange used.
With a proper connector a new Exchange implementation different from Bitso could be plugged-in and used with the same application and strategies.

### stpc-exchange

Basic interfaces that represent a generic exchange. (Actually this module contains only the method needed for the exercise).

### stpc-exchange-bitso

Bitso implementation for module `stpc-exchange`.

An effort has been made to **not** use Spring nor any big framework in this library so it is kept as lightweight as possible, so it can be used in another projects reducing risk of version incompatibilities or conflicts. (Some reference articules: [DZone](https://dzone.com/articles/kill-your-dependencies-javamaven-edition) or [Jooq](https://blog.jooq.org/2016/08/11/all-libraries-should-follow-a-zero-dependency-policy/))

It is intended to be highly concurrent trying to use fine-grained locks as specific as possible.


### stpc-app

It is a SpringBoot application which starts a web server and publish two REST Endpoints:

- `/api/v1/strategies/`: Strategy management. It can reconfigure or show nice statistics about a running strategy (see in the current app example: (http://localhost:8080/api/v1/strategies/contrarian1)). This REST Controller would normally also create more Strategy *instances* of a given type on http POST method, etc...
- `/api/v1/exchange/`: Can be requested about best asks, best bids and last trades.

It also contains the FE files and the "Strategy framework" (with just a ContrarianStrategy as Strategy type)


---

## Checklist

| Feature | File name | Method name |
| ------- | --------- | ----------- |
| Schedule the polling of trades over REST | [NewTradesNotifier.java](https://bitbucket.org/martinez-ruben/sonar-programming-challenge/src/master/stpc-exchange-bitso/src/main/java/net/rubenmartinez/stpc/exchange/bitso/trade/helper/NewTradesNotifier.java) | start |
| Request a book snapshot over REST | [NewOrderBookSupplier.java](https://bitbucket.org/martinez-ruben/sonar-programming-challenge/src/master/stpc-exchange-bitso/src/main/java/net/rubenmartinez/stpc/exchange/bitso/orderbook/NewOrderBookSupplier.java) | getNewFullBook |
| Listen for diff-orders over websocket | [BitsoWebsocketClient.java](https://bitbucket.org/martinez-ruben/sonar-programming-challenge/src/master/stpc-exchange-bitso/src/main/java/net/rubenmartinez/stpc/exchange/bitso/api/websocket/BitsoWebsocketClient.java) | onMessage |
| Replay diff-orders | [ReplayQueueOrderBookKeeper.java](https://bitbucket.org/martinez-ruben/sonar-programming-challenge/src/master/stpc-exchange-bitso/src/main/java/net/rubenmartinez/stpc/exchange/bitso/orderbook/ReplayQueueOrderBookKeeper.java) | ResetBookTask.run |
| Use config option X to request recent trades | [ExchangeRestController.java](https://bitbucket.org/martinez-ruben/sonar-programming-challenge/src/master/stpc-app/src/main/java/net/rubenmartinez/stpc/app/controller/ExchangeRestController.java) | getLastTrades |
| Use config option X to limit number of ASKs displayed in UI | [ExchangeRestController](https://bitbucket.org/martinez-ruben/sonar-programming-challenge/src/master/stpc-app/src/main/java/net/rubenmartinez/stpc/app/controller/ExchangeRestController.java) | getBestAsks |
| The loop that causes the trading algorithm to reevaluate | [ContrarianStrategy](https://bitbucket.org/martinez-ruben/sonar-programming-challenge/src/master/stpc-app/src/main/java/net/rubenmartinez/stpc/app/strategy/implementations/contrarian/ContrarianStrategy.java) | reevaluatePastTrades |
