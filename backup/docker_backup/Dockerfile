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