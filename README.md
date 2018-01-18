
Steps to reproduce



1. Bring up zookeeper, kafka and flink

       docker-compose up -d
    
    
2. Build the job

       mvn clean package
 
3. Deploy the job

       docker exec -i -t flink /opt/flink/bin/flink run -q -d /app/flink-app-1.0.jar

4. View the flink console output 

       docker-compose logs -f -t flink
       
5. Send some messages to the kafka topic

   Leave the console output running then in another terminal start the console producer 

       docker exec -i -t kafka /opt/kafka/bin/kafka-console-producer.sh --topic mytopic --broker-list kafka:9092
       
   Enter the following lines of text
   
       message 1
       message 2
   
   If you look at your other terminal these messages should be in the console output
   
6. Save the state of the job

    Open yet another terminal and execute
 
       docker exec -i -t flink /opt/flink/bin/flink list
       
   to find out the job id.
   
   and then cancel the job with a savepoint  
    
       docker exec -i -t flink /opt/flink/bin/flink cancel -s <job id>
       
   The output wil indicate where the savepoint has been created
       
7. redeploy the jon from a savepoint
       
       docker exec -i -t flink /opt/flink/bin/flink run -q -d -s <savepoint> /app/flink-app-1.0.jar

8. Send more messages and make the job blow up and restart itself

   Switch back to the terminal that runs the kafka console producer and enter the following lines of text
   
       message 3
       message 4
       message 5
       implode


       
The implode message will make the flink job throw an exception and restart itself but it will start processing
messages from the save point! 

Because this app does not do checkpointing when the savepoint was created might be a long long long time ago by the time
an exception occurs
   