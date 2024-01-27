
Step 1: Create a Dockerfile
```
# Use the official PostgreSQL base image with the desired version
ARG PG_MAJOR=15
FROM postgres:$PG_MAJOR
ARG PG_MAJOR

# Install necessary dependencies for building pgvector extension
RUN apt-get update && \
    apt-get install -y \
        build-essential \
        postgresql-server-dev-$PG_MAJOR \
        git

# Clone the pgvector repository and build the extension
RUN git clone https://github.com/ankane/pgvector.git && \
    mv pgvector /tmp/pgvector && \
    cd /tmp/pgvector && \
    make clean && \
    make OPTFLAGS="" && \
    make install && \
    rm -rf /tmp/pgvector

# Set up the PostgreSQL database credentials
ENV DB_USER=vectordb
ENV DB_PASSWORD=vectordb
ENV DB_NAME=vectordb

# Copy the initialization script to the Docker container
COPY init.sql /docker-entrypoint-initdb.d/

# Expose the default PostgreSQL port
EXPOSE 5432

# Start the PostgreSQL server
CMD ["postgres"]


```

Step 2: Build the Docker image
```
docker build -t dbvector_postgres .
```

Step 3: Run the Docker container
```
docker run -d -p 5432:5432 --name dbvector_postgres -e POSTGRES_USER=vectordb -e POSTGRES_PASSWORD=vectordb -e POSTGRES_DB=vectordb dbvector_postgres
```

In the updated Dockerfile and `docker run` command, we added the environment variables `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB` with their respective values. These environment variables will set the credentials for the PostgreSQL database in the Docker container.

Now when you connect to the PostgreSQL database from your Python code, you'll use the following connection details:

```python
import psycopg2

# Establish connection to the PostgreSQL Docker container
conn = psycopg2.connect(
    host="localhost",
    port=5432,
    user="vectordb",
    password="vectordb",
    database="vectordb"
)

# Continue with the rest of your code
```
# Create a cursor object
cursor = conn.cursor()

# Execute a test query
cursor.execute("SELECT version();")

# Fetch the result
result = cursor.fetchone()
print("Connection successful!")
print("PostgreSQL version:", result[0])

# Close the cursor and connection
cursor.close()
conn.close()
```
You got
```
Connection successful!
PostgreSQL version: PostgreSQL 16.1 (Debian 16.1-1.pgdg120+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 12.2.0-14) 12.2.0, 64-bit
```
