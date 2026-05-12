# Spring Boot EC2 Demo Project

## Run Project
```bash
mvn clean install
mvn spring-boot:run
```

## Build Jar
```bash
mvn clean package
```

Jar file will be generated inside:
```bash
target/
```

## API Endpoints

### Hello API
GET /

### Create Employee
POST /employees

Example JSON:
```json
{
  "name":"Madhav",
  "role":"Java Developer",
  "salary":50000
}
```

### Get All Employees
GET /employees

### Get Employee By Id
GET /employees/{id}

### Update Employee
PUT /employees/{id}

### Delete Employee
DELETE /employees/{id}
```
