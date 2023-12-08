# weibo networks analysis

## Introduction

A system for analysis of **topics**, **blog posts**, and **user profiles** on Weibo.

Features:

- Weibo Topic Blog Post Search: Retrieves the top 50 pages, nearly 1000 blog posts, based on user-input keywords, ranked by popularity.

- Weibo Topic Analysis: Users can add related topics to the analysis task queue, which the backend asynchronously processes (using Celery).
  
  - Topic Task List: All tasks in the current system, including those under analysis, completed, and those encountering exceptions.
  
  - Top Blog Posts on Topic: Displays the top ten most popular blog posts under the topic.
  
  - Topic Popularity: Shows the topic's popularity over recent days, a month, and three months.
  
  - Topic Word Cloud: Analyzes words associated with the topic based on all blog post content gathered.
  
  - Simplified Topic Propagation Graph: Represents the topic's spread using a directed graph.
  
  ![illustration](https://github.com/Faker-lz/Topic_and_user_profile_analysis_system/blob/master/doc/illustration/%E8%AF%9D%E9%A2%98%E4%BB%BB%E5%8A%A1%E5%88%97%E8%A1%A8.png)

- Blog Post Analysis: Detailed analysis of the top ten popular blog posts within a topic.
  
  - Blog Post Details: Displays the text of the blog post and rudimentary information about the user who posted it.
  
  - Key Nodes in Blog Post Spread: Lists nodes playing a significant role in spreading the blog post, often those that generate substantial attention post-forwarding.
  
  - Blog Post Propagation Tree: The forwarding propagation tree of the blog post.
  
  - Blog Post Comment Word Cloud: Words associated with comments on the blog post.
  
  - Blog Post Topic Analysis: Themes in the **comments** under the blog post and their proportions.
  
  - Blog Post Hot Forwarding: Displays content with high attention in forwarding.
  
  ![image](https://github.com/AndyZhang1999/Meida_network_via_weibo/assets/90740478/77859f51-fe4f-4784-bc73-9b033b48d24a)


- User Profiling: Tags users based on the clustering themes of their Weibo texts under **different topics**, progressively refining the user profile.
  
  ![illustration](https://github.com/Faker-lz/Topic_and_user_profile_analysis_system/blob/master/doc/illustration/%E8%AF%9D%E9%A2%98%E5%86%85%E7%94%A8%E6%88%B7%E6%A0%87%E7%AD%BE%E5%8F%8A%E5%85%B7%E4%BD%93%E4%BC%A0%E6%92%AD%E5%85%B3%E7%B3%BB.png)

## Design

* Overview Design: [Frontend Low-Fidelity Prototype Design](https://modao.cc/app/096f66e13ccb38c83e73e67f3fbdb091526d900b?simulator_type=outside_artboard)

* Detailed Design:
  
  * Architectural Design: The project adopts a BS architecture for frontend-backend separation. The frontend uses the Vue framework, implemented following the [Frontend Low-Fidelity Prototype Design](https://modao.cc/app/096f66e13ccb38c83e73e67f3fbdb091526d900b?simulator_type=outside_artboard), while the backend is developed with Python's [FastAPI](https://fastapi.tiangolo.com/zh/) framework. The integration with the distributed asynchronous task handling framework [Celery](https://www.celerycn.io/ru-men/zhong-jian-ren-brokers/shi-yong-redis) enables decoupling of task publishing and execution, facilitating fast and asynchronous processing of topic analysis tasks.
  
  * Algorithms Involved:
    
    * GsdmmCluster based on MGP for short text clustering analysis of blog posts, applied to cluster analysis of comments on a particular post.
    
    * LDA for clustering analysis within the **public opinion field** of topics.
    
    * Single_Pass for clustering analysis within the topic **public opinion field**.
  
  * Routing and Database Design Documents: Located in `\doc\design`

## Configuration

* Backend Configuration:
  
  * Modify the following settings in the file `\code\back_end\config\config_class`:
    
    ```python
    class AppConfig(BaseSettings):
        """
        Configuration for fastapi_app launch
        """
        HOST: str = '127.0.0.1'
        PORT: int = 81
    
    
    class WeiBoConfig(BaseSettings):
        """Weibo crawler API configuration"""
        BASEPATH: str = 'http://127.0.0.1:8000'
    ```
  
  * Modify Celery-related configuration in the file `\code\back_end\celery_task\config\task_config_class.py`:
    
    ```python
    from pydantic import BaseSettings
    
    class CeleryConfig(BaseSettings):
        """
        Configuration for celery launch
        """
        BROKER = 'redis://localhost:6379/0'
        BACKEND = 'redis://localhost:6379/1'
    
    
    class MongoConfig(BaseSettings):
        """
        Mongo Configuration
        """
        HOST: str = '127.0.0.1'
        PORT: int = 27017
        DB_NAME: str = 'test'
        # various database names for tasks and data
        ...
    ```

* Weibo Crawler Microservice Configuration:
  
  Please refer to the configuration in `\code\weibo_crawler\README.md`

## Launch

* Backend
  
  * Install Redis database
  
  * Install all dependencies required by the backend as listed in the project's `requirements.txt`
    
    > It is highly recommended to build a virtual environment as guided in `doc\后端部署指南.docx`.
  
  * Run `\code\back_end\main.py` to start the FastAPI app
  
  * Launch Redis database
  
  * Navigate to `\code\back_end` and execute `celery -A celery_task.celeryapp worker --loglevel=info -P threads` to start Celery

* Frontend
  
  * Navigate to `\code\back_end`, follow the `README.md` file to install dependencies.
  
  * Execute `npm install` to set up the Vue project.
  
  * Run `npm run server` to launch the frontend.

* Weibo Crawler
  
  Navigate to `\code\weibo_crawler`, configure as instructed, then run `weibo_curl_api.py`
