
This program was built as part of a coding challenge. The purpose is to implement Drone Scheduler and calculate NPS score based on delivery time.

## Building and Running the application

#### The application has been built and tested in a `Java 10` runtime. Ensure that Java 10 is installed on your machine prior to downloading the repository.

The application can be downloaded using the following command (sample input file droneOrderInput.txt is provided)

- `git clone https://github.com/skurdekar/DroneDelivery.git`


#### Gradle 4.9+ is needed to build and run the application (specifically the support for arguments that are passed to gradle has been added only in 4.9+). You can install gradle (https://gradle.io) or use the wrapper `gradlew` packaged with the application source.

#### Running behind corporate proxy
The corporate proxy information needs to be stored in gradle.properties file
On Unix based systems this is ~/.gradle/gradle.properties file

```
systemProp.http.proxyHost=<proxhost>
systemProp.http.proxyPort=<proxyport>
systemProp.http.proxyUser=<userid>
systemProp.http.proxyPassword=<pass>
systemProp.https.proxyHost=<proxyhost>
systemProp.https.proxyPort=<proxyport>
systemProp.https.proxyUser=<userid>
systemProp.https.proxyPassword=<pass>
```

The application is packaged with the Gradle Build Tool. It can be built using the following command
- `./gradlew clean build`

The application can be run as follows (using sample or user provided input)
- `./gradlew run --args='-f <filepath> -nd <numdrones> -no <numorders> -g'`

##### `-f` path of the input file (mandatory)
##### `-g` generate random input file at specified path with the below specified parameters (optional)
##### `-no` number of simulated orders in input file (optional 1-500)
##### `-nd` number of drones for parallel processing (1-15)
##### `-bb` max number of blocks to traverse for sample data set (1-20)

The number of drones is defaulted to 1 if no parameter is provided. The maximum number of drones that can be run in parallel are 15.
- `./gradlew run --args='-f droneOrderInput.txt -nd 3'`

Input file lines starting with `#` are considered to be comments and ignored.

The build can be cleaned as follows
- `./gradlew clean`

Alternatively the application can be packaged as a fat jar
- `./gradlew clean build fatJar`

Once packaged as a fat jar it can be run as a standalone java application
- `java -classpath build/libs/dronedelivery-all-1.0-SNAPSHOT.jar com.dronedelivery.DroneScheduler -f <inputFilePath> -nd numdrones`

## Example Output

The application implements logging using the log4j library. The log configuration is available in log4j.properties file in resources directory. Currently the application outputs log4j formatted messages set to DEBUG level (in log4j.properties file). Both the console and file output are set to the same level. Sample output is included below. The output file is presented as the last line of the output and is not configurable at this time.

```
    output file path: /Users/skurdekar/droneDeliveryOut/
    location: N11W5 delivery time: 724.98
    OrderId: WM001 Min Delivery Time: 01:00:15
    location: S3E2 delivery time: 216.33
    OrderId: WM002 Min Delivery Time: 00:51:41
    location: N7E50 delivery time: 3029.26
    OrderId: WM003 Min Delivery Time: 01:18:39
    location: N11E5 delivery time: 724.98
    OrderId: WM004 Min Delivery Time: 00:12:05
    OrderId: WM002 NPS: 10.0
    startProcessing: Processed Order: OrderId: WM002 PlaceTime: 05:11:55 MinDeliveryTime: 00:51:41 DispatchTime: 06:00:00 DeliveryTime: 06:03:36 ReturnTime(+1): 06:07:12
    OrderId: WM001 NPS: 8.875555
    startProcessing: Processed Order: OrderId: WM001 PlaceTime: 05:11:50 MinDeliveryTime: 01:00:15 DispatchTime: 06:07:13 DeliveryTime: 06:19:18 ReturnTime(+1): 06:31:23
    OrderId: WM004 NPS: 10.0
    startProcessing: Processed Order: OrderId: WM004 PlaceTime: 06:11:50 MinDeliveryTime: 00:12:05 DispatchTime: 06:31:24 DeliveryTime: 06:43:29 ReturnTime(+1): 06:55:34
    OrderId: WM003 NPS: 7.762778
    startProcessing: Processed Order: OrderId: WM003 PlaceTime: 05:31:50 MinDeliveryTime: 01:18:39 DispatchTime: 06:55:35 DeliveryTime: 07:46:04 ReturnTime(+1): 08:36:33
    calculateNPS: NPS: 75
    DroneDelivery: successfully wrote output to /Users/skurdekar/droneDeliveryOut/droneDeliveryOut.txt
```
#### Output file(not configurable) is located at `${userhome}/droneDeliveryOut/droneDeliveryOut.txt`

#### Rejects file(not configurable) is located at `${userhome}/droneDeliveryOut/droneDeliveryRejects.txt`

## Unit Tests
- `./gradlew clean test`

See DroneDeliveryAppTest class for test examples. 
Test reports are generated in build/reports/tests/test/classes directory
All tests are run everytime a gradle clean build command is run. Sample test output is included below

```
DroneDeliveryAppTest > testDispatchTime STARTED
Executing test testDispatchTime [DroneDeliveryAppTest] with result: SUCCESS

DroneDeliveryAppTest > testDispatchTime PASSED

DroneDeliveryAppTest > testInvalidOrderParam STARTED
Executing test testInvalidOrderParam [DroneDeliveryAppTest] with result: SUCCESS

DroneDeliveryAppTest > testInvalidOrderParam PASSED

DroneDeliveryAppTest > testTooFar STARTED
Executing test testTooFar [DroneDeliveryAppTest] with result: SUCCESS

DroneDeliveryAppTest > testTooFar PASSED
```

## Architecture Logic & Assumptions

### Current Architecture

![Screenshot](https://github.com/skurdekar/DroneDelivery/blob/master/architecture/DroneCurrent.png)

### Logic used to Schedule the Drone and pick the next Order
The orders are scheduled based on least time to delivery from the time of drone facility opening. The dispatch time starts when the facility opens. In case an order is placed after the facility opens, but takes lesser time to deliver than an order already in queue it will be scheduled for delivery ahead of the older order (with some conditions). If no orders are found with above rule, the earlier placed order will be taken for delivery. The time to delivery is the diagonal distance (hypotenuse) of the triangle with NS and EW co-ordinates from the delivery center. Based on the example provided it was determined that the Drone can travel diagonally to deliver an order.

### Drone Operating Area
Since the Drone dispatch center is open for 16 hours every day (from 6 am to 10 pm) and the Drone speed is 1 block (horizontal or vertical) per minute we have to ensure the Drone gets back in 16 hours. This limits the operating distance to an area that takes less than 8 hours to and fro for delivery. We will limit the circular area to 480 radial blocks (giving us a diagonal to be back in 16 hours for every delivery). Any location that results in a diagonal bigger than 480 blocks will be deemed unreachable and the order will not be processed. Options considered for delivery area were Square, Rectangular and Circular. Circular shape gives us the biggest processing area and is hence used for the solution.

### Invalid parameters in input file
If the file contains bad data the order will be rejected and count against NPS calculation as a detractor with the worst score. Examples of bad data include invalid or incomplete parameters based on the specifications provided. Bound checks have not been performed and all numbers are limited to integer bounds.

### NPS Calculation
NPS Calculation will be done using the following formula
`promoterCount/sampleSize*100 - detractorCount/sampleSize*100`
Passive responses (7,8) will count towards sample size. Samplesize = promoterCount + detractorCount + passiveCount
```For the provided sample input, the NPS starts at -45 for 1 drone processor and tops off at 96 for 10 drones. Adding additional drones significantly improves the NPS (per drone added).```

### Error handling
There is minimal error handling implemented in the application which is in no way indicative of how an application should handle edge cases, bad data and erroneous conditions in production. 

### Assumption: Handling time at dispatch and delivery
Based on the example provided there is no delay in depositing an order at its destination or loading items at the dispatch facility. The only delay in the processing is 1 second delay between returning of the Drone and next dispatch. The application builds on the sample output provided for its calculations but this is not a real world scenario where there will be some delays.

### Assumption: Deliveries that cannot be processed the same day will be rejected
Prior to scheduling delivery the scheduler ensures that the Drone can be back before the operating center closes (10 pm). Any orders that cannot be processed in time will be rejected and stored in a reject file droneDeliveryRejects.txt (in the same directory as output file). All rejects will be considered incomplete orders and can be set for manual process the next day. The handling of rejects processing is beyond the scope of this solution.

### Assumption: Solution will not handle rolling over undelivered orders to next day
For NPS calculation rejected orders will be considered to have a 0 NPS Score. Handling of rejected orders is beyond scope of this application. While it is understood that within the NPS calculation window, orders can still be delivered next day by 8 am that consideration has not been used.

### Assumption: Drone Operability
I have made a few assumptions which may create some complications in real life scenario. Drone has enough battery life to run for 16 hours. Wind and payload do not affect drone speed. Drone is able to carry only one package at a time and has to return to facility to reload.

## Next Steps

### Possible Future Architecture

![Screenshot](https://github.com/skurdekar/DroneDelivery/blob/master/architecture/DroneFuture.png)

- Security - Authentication and Authorization

- The problem poses a significant scheduling challenge at real world scale. Drones operating areas will need to be shared with  overlapping circular regions where multiple Drones can operate in unision. Shared drones will need to work off of concurrent order queues. Calculation and sizing can be based on how much a drone can travel in a particular day. Delivery prioritization based on shiping type should also be taken into account.

- Enhanced unit tests will need to be added for border and edge case scenarios.

- OrderProcessor can implement a service interface for remote interaction with other Drone Schedulers. Queue information can hence be shared.

- Create microservice APIs for admin as well as customers of the Drone Delivery Service. Add APIs to query the process queue information as well as NPS on Order Processor.

- Edge case scenarios and handling bad data needs to be done in a comprehensive manner.
