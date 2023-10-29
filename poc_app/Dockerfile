# official python image
FROM python:3.10

# Set the working directory to /app
WORKDIR /app

# Copy the requirements file into the container at /app
COPY requirements.txt .

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Copy the rest of the application code into the container at /app
COPY ./src ./src

# Expose port 8000 for the application
EXPOSE 8000

# Run the command to start the application
CMD ["uvicorn", "src.main:app", "--port", "8000", "--host", "0.0.0.0"]
