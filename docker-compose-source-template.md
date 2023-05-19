# Docker Compose Template for Stack Deployment

####  MERN Stack
```yaml
version: '3'

services:
  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
    volumes:
      - ./mongo-data:/data/db
  api:
    build: ./api
    restart: always
    ports:
      - 5000:5000
    environment:
      - MONGODB_URI=mongodb://mongo:27017/mern
  client:
    build: ./client
    restart: always
    ports:
      - 3000:3000
    environment:
      - REACT_APP_API_URL=http://api:5000
```
#### MEVN Stack
```yaml
version: '3'

services:
  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
    volumes:
      - ./mongo-data:/data/db
  api:
    build: ./api
    restart: always
    ports:
      - 3000:3000
    environment:
      - MONGODB_URI=mongodb://mongo:27017/mevn
  client:
    build: ./client
    restart: always
    ports:
      - 8080:8080
```

#### MEAN Stack
```yaml
version: '3'

services:
  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
    volumes:
      - ./mongo-data:/data/db
  api:
    build: ./api
    restart: always
    ports:
      - 3000:3000
    environment:
      - MONGODB_URI=mongodb://mongo:27017/mean
  client:
    build: ./client
    restart: always
    ports:
      - 4200:4200
```
#### Flask
```yaml
version: '3'

services:
  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: flask
    ports:
      - 5432:5432
  web:
    build: .
    restart: always
    environment:
      - DATABASE_URL=postgres://admin:admin@db:5432/flask
    ports:
      - 5000:5000
```

#### Django
```yaml
version: '3'

services:
  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: django
    ports:
      - 5432:5432
  web:
    build: .
    restart: always
    environment:
      - DATABASE_URL=postgres://admin:admin@db:5432/django
    ports:
      - 8000:8000
```

#### Laravel
```yaml
version: '3'

services:
  db:
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel
    ports:
      - 3306:3306
    volumes:
      - ./mysql-data:/var/lib/mysql
  app:
    build: .
    restart: always
    environment:
      DB_CONNECTION: mysql
      DB_HOST: db
      DB_PORT: 3306
      DB_DATABASE: laravel
      DB_USERNAME: root
      DB_PASSWORD: root
    ports:
      - 8000:8000
```
