# Application Architecture

## Data

- Prices: date, close, hihg, low, open, volume, adjClose, adjHigh, adjLow, adjOpen, adjVolume, divCash, splitFactor

1. Write a whole series once
2. Read the whole series many times
3. Every day, run a cron job to appendd each series with a new price point
4. At times, delete bad data

```json
{
  "ticker": "...",
  "data": {
    "date": ["date1", "dateN"],
    "open": ["price1", "priceN"]
  }
}
```

## OpenAI Integration

- [Open AI Completion Stream](https://stackoverflow.com/questions/76137987/)

## AWS SQS (Simple Queue Service)

SQS is not strictly needed for triggered a Lambda function (can invoke directly from an API Gateway, S3 event, or other event sources). However, it can be beneficial in some scenarios, especially when you're dealing with long-running operations:

1. **Asynchronous processing and decoupling**: Decouple the request from the processing. The client that is creating the job does not have to wait for the job to complete. The job can be put in the SQS queue and processed separately.
   - In contrast, if you invoke a Lambda synchronously (default invocation type), the function runs and the service waits for the function to complete. If the function takes 5 minutes to complete for example, the client making the request will need to wait for the same duration for a response.
   - Once jobs are put in the SQS queue, getting the result of the job back to the client might require additional architecture considerations, depending on what you want to do with the result. If the client needs to know the result of the job, you might need to store the result in a database that the client can poll, or use a real-time notification system like AWS SNS or WebSocket to notify the client when the job is done.
2. **Throttling and managing peak load**: Lambda has concurrency limits. If you have a spike in events triggering your Lambda function, using SQS as a buffer can help manage and smooth out those peaks. The messages (tasks) will remain in the queue until Lambda is ready to process them, preventing overloading of your function and potential throttling.
3. **Retries and Dead Letter Queues (DLQ)**: SQS allows automatic retries if your function fails to process a message. After a certain number of retries, the message can be moved to a DLQ for further examination. This makes error handling easier and more robust.
4. **Preserving order of messages**: If order of processing is important, SQS offers FIFO queues.

## Polling a database (client side)

1. When client submits a job, your backend creates a record in the database for that job, including a unique job ID, status of the job, and any other relevant information.
2. The client receives the job ID as an immediate response.
3. The job gets placed in the SQS queue and is eventually processed by the Lambda function. Once the job is complete, the Lambda function updates the job's record in the database and stores any results of the job.
4. Meanwhile, the client periodically sends requests to your backend, asking for the status of the job with the given job ID.
5. Once the status is "complete", the client can retrieve the results and stop polling.

## Python

- **FastAPI**: Modern, fast (high-performance) web framework for building APIs with Python 3.6+. Designed to be easy to use, while also enabling high performance with the ability to handle asynchronous requests.
- **asyncpg**: Database interface library designed specifically for PostgreSQL and Python/asyncio. Designed to be fast and efficient, making it a great choice for applications that need to handle a large number of concurrent database connections.
- **ujson** (UltraJSON): JSON library for Python. Designed to be ultra-fast and efficient, making it a good choice for applications that need to handle a lot of JSON data.
- **Gunicorn** (Green Unicorn): Python WSGI (Web Server Gateway Interface) HTTP Server for UNIX. It's a pre-fork worker model, which means that it forks multiple worker processes to handle incoming requests. This makes it a good choice for applications that need to handle a high volume of HTTP requests.

## 3D modeling, animation, and game development

- **Blender**: Free and open-source 3D computer graphics software toolset. It's used for creating animated films, visual effects, art, 3D printed models, motion graphics, interactive 3D applications, and computer games. Blender's features include 3D modeling, UV unwrapping, texturing, raster graphics editing, rigging and skinning, fluid and smoke simulation, particle simulation, soft body simulation, sculpting, animating, match moving, rendering, motion graphics, video editing, and compositing.
- **Maya**: Autodesk Maya is a professional-grade 3D modeling and animation software. It's used in the film and TV industry, as well as for video games. Maya offers a comprehensive suite of tools for 3D creation, including modeling, texturing, rigging, animation, and rendering. It's known for its flexibility and precision, making it suitable for complex simulations and effects.
- **Unity**: Unity is a cross-platform game engine that is used to develop and publish video games for computers, consoles, mobile devices, and websites. Unity supports both 2D and 3D development, and it includes a variety of tools for designing and building games, including a physics engine, audio management, and scripting capabilities. Unity also supports virtual reality and augmented reality content.

Blender and Maya are primarily used for creating and animating 3D models, while Unity is used to bring these models into an interactive environment.

## Links

- [AWS SQS + Lambda Setup Tutorial - Step by Step](https://www.youtube.com/watch?v=xyHLX1dUwuA)
- [GitHub as a CMS](https://dev.to/nuro/using-github-as-a-cms-2n28)
