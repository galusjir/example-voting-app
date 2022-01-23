FROM maven:3.6.1-jdk-8-alpine

WORKDIR /app

COPY . .

RUN mvn package -DSkipTests && \ 
    mv target/worker-jar-with-dependencies.jar /run/worker.jar && \
    rm -rf /app/*

#EXPOSE 

CMD java -jar /run/worker.jar
 
