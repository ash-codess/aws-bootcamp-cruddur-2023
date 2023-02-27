<img width="1584" alt="W1-B(1)" src="https://user-images.githubusercontent.com/123767474/220916748-3a40d075-2b7c-4087-b016-0ab0e390bd6c.png">


# Week 1 — App Containerization


![GitHub last commit](https://img.shields.io/github/last-commit/ash-codess/aws-bootcamp-cruddur-2023)

<details>
<summary>
Youtube live takeaway
</summary>
This week we started with docker. As a beginner it was overwhelming amount of information but now i knew what needs to be done and i decided to give first two days to understand more about docker.

<img width="582" alt="docker-arch" src="https://user-images.githubusercontent.com/123767474/221354749-c4ca3cef-19d9-4c5f-94d1-0c50327478c5.png">

I started with the free docker for beginners course by Kodekloud. The first few section of the course was enough to understand the basics of docker. I documented what i learned in my personal blog
[Link](https://ash-codes.tech/posts/docker-101/).<br>
Next step was to install docker locally on my system and run some images myself. I did so using convenience script (steps mentioned in blog).
When i felt comfortable enough to understand what was going on, I proceeded with this week's required work!

</details>

---

<details>
<summary>Required Homework</summary>

1. Setting up docker in gitpod
   - Install the docker extension in gitpod

<br>

2. Setting up docker in backend
   - Created a Dockerfile in backend-flask with following code:
        ```sh
        FROM python:3.10-slim-buster
        
        #Inside container
        WORKDIR /backend-flask
        
        #Outside container -> Inside container (contains libraries to run the app)
        COPY requirements.txt requirements.txt

        #Inside container (installing python libraries used for the app)
        RUN pip3 install -r requirements.txt
        
        #Outside container -> inside directory
        COPY . .
        
        #set env vars  (inside container)
        ENV FLASK_ENV=development
        
        EXPOSE ${PORT}

        #python3 -m flask run --host=0.0.0.0 --port=4567
        CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
        ```

    - Install the requirements.txt: 
        ```sh
        cd backend-flask
        pip3 install -r requirements.txt
        ``` 

    - Setting up env vars:
        ```sh
        export FRONTEND_URL="*"
        export BACKEND_URL="*"
        ```    

    - To get the server running on port 4567, run the following:
        ```sh
        python3 -m flask run --host=0.0.0.0 --port=4567
        ```
         We will get back json file on successful setup! Make sure the port 4567 is unlocked and url is modifies at the end to  "api/activities/home"

    - Unset the env vars for now (because dockerfile might not take the environment variables already set in the system):
        ```sh
        unset FRONTEND_URL="*"
        unset BACKEND_URL="*"

        #To make sure it's gone, run this:
        env | grep = _URL
        ```

    - To build the image:
        ```sh
        #navigate back to root directory
        cd .. 
        docker build -t  backend-flask ./backend-flask
        ```   
    - To run the container:
        ```sh
        FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
        ```
      This gave a 404 error, on debugging we found that env vars weren't set in the container (we used attached shell to figure this out)
    
    - To remove this error, we will run this modified  version of the command:
        ```sh
        docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
        ```
        We will get back json file on successful setup! Make sure the port 4567 is unlocked and url is modifies at the end to  "api/activities/home"  

    -  Run docker ps it will show up all the images which are running     

<br> 

3. Setting up docker in frontend

    - Installing the requirements:        
        ```sh
        cd  frontend-react-js
        npm i
        ```

    - Create a Dockerfile in /frontend-react-js directory with following code:
        ```sh
        FROM node:16.18
        ENV PORT=3000
        COPY . /frontend-react-js
        WORKDIR /frontend-react-js
        RUN npm install
        EXPOSE ${PORT}
        CMD ["npm", "start"]
        ```

    - Go to root directory and build the image:
        ```sh
        docker build -t frontend-react-js ./frontend-react-js
        ```

    - Finally run the container:
        ```sh
        docker run -p 3000:3000 -d frontend-react-js
        ```

4. Running multiple containers with docker compose
    - Go to root directory and make a new file called  docker-compose.yml and paste in thr following code:


        ```sh
        version: "3.8"
        services:
            backend-flask:
                environment:
                    FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
                    BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
                build: ./backend-flask
                ports:
                    - "4567:4567"
                volumes:
                    - ./backend-flask:/backend-flask
            frontend-react-js:
                environment:
                    REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
                build: ./frontend-react-js
                ports:
                    - "3000:3000"
                volumes:
                    - ./frontend-react-js:/frontend-react-js
            # the name flag is a hack to change the default prepend folder
            # name when outputting the image names
        networks: 
            internal-network:
            driver: bridge
            name: cruddur
        ```

    - Finally to run it, hit right-click on docker-compose.yaml and hit compose up and the ports should be activated. If you did  everything correctly you should see the following output
  
        ![week1_setup-proof-of-work](https://user-images.githubusercontent.com/123767474/220918968-61d5200c-4d5c-4ff6-a3d6-79b007cee490.png)

  

5. Notification feature for the app
    - Go to the frontend-react directory and run: npm i
    - Go back to root directory then Docker up to spin up the ports
    - Modify backend-flask/openai.yaml file with:
        ```sh
        /api/activities/notifications:
        get:
          description: 'Return a feed of activity for all of those that I follow'
        tags:
            - activities
        parameters: []
        responses:
            '200':
            description: Returns an array of activities
            content:
                application/json:
                schema:
                    type: array
                    items:
                        $ref: '#/components/schemas/Activity'
        ```
    - Go to backend-flask/app.py and add in an endpoint (below api/activities/home route):
        ```sh
        @app.route("/api/activities/notifications", methods=['GET'])def data_notifications():
            data = NotificationsActivities.run()
            return data, 200
        ```
    - Create a new file called notifications_activities.py at location backend-flask/services and add mock data:
        ```sh
        from datetime import datetime, timedelta, timezone
        class NotificationsActivities:
            def run():
                now = datetime.now(timezone.utc).astimezone()
                results = [{
                    'uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
                    'handle':  'space cadet',
                    'message': 'I am an astro',
                    'created_at': (now - timedelta(days=2)).isoformat(),
                    'expires_at': (now + timedelta(days=5)).isoformat(),
                    'likes_count': 5,
                    'replies_count': 1,
                    'reposts_count': 0,
                    'replies': [{
                    'uuid': '26e12864-1c26-5c3a-9658-97a10f8fea67',
                    'reply_to_activity_uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
                    'handle':  'worf',
                    'message': 'this post has no honor!',
                    'likes_count': 0,
                    'replies_count': 0,
                    'reposts_count': 0,
                    'created_at': (now - timedelta(days=2)).isoformat()
                }],
            }
            ]
        return results
        ```

    - Add it to the app.py file 
        ```sh
        from services.notifications_activities import *
        ```
    - Modify the 4567 port url with: /api/activities/notifications.
    We should get back json data
    Our backend setup is  now done!

    - For frontend lets modify the App.js file in /frontend-react-js/src/
        ```sh
        import NotificationsFeedPage from './pages/NotificationsFeedPage';

        #Add in the path below
        {
        path: "/notifications",
        element: <NotificationsFeedPage />
        }
        ```
    - Go to /pages folder and create a new file called NotificationsFeedPage.js and paste in the code from homefeed.js file with necessary changes.

        ```sh
        import './NotificationsFeedPage.css';
        import React from "react";

        import DesktopNavigation  from '../components/DesktopNavigation';
        import DesktopSidebar     from '../components/DesktopSidebar';
        import ActivityFeed from '../components/ActivityFeed';
        import ActivityForm from '../components/ActivityForm';
        import ReplyForm from '../components/ReplyForm';

      # [TODO] Authenication
        import Cookies from 'js-cookie'

        export default function NotificationsFeedPage() {
            const [activities, setActivities] = React.useState([]);
            const [popped, setPopped] = React.useState(false);
            const [poppedReply, setPoppedReply] = React.useState(false);
            const [replyActivity, setReplyActivity] = React.useState({});
            const [user, setUser] = React.useState(null);
            const dataFetchedRef = React.useRef(false);

            const loadData = async () => {
                try {
                    const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/activities/notifications`
                    const res = await fetch(backend_url, {
                    method: "GET"
                });
                let resJson = await res.json();
                    if (res.status === 200) {
                        setActivities(resJson)
                    } else {
                        console.log(res)
                    }
                    } catch (err) {
                        console.log(err);
                    }
                };

            const checkAuth = async () => {
                console.log('checkAuth')
            # [TODO] Authenication
            if (Cookies.get('user.logged_in')) {
                setUser({
                display_name: Cookies.get('user.name'),
                handle: Cookies.get('user.username')
            })
            }
        };

            React.useEffect(()=>{
            #prevents double call
                if (dataFetchedRef.current) return;
                dataFetchedRef.current = true;

                loadData();
                checkAuth();
            }, [])

            return (
                <article>
                <DesktopNavigation user={user} active={'notifications'} setPopped={setPopped} />
                <div className='content'>
                    <ActivityForm  
                        popped={popped}
                        setPopped={setPopped} 
                        setActivities={setActivities} 
                    />
                    <ReplyForm 
                        activity={replyActivity} 
                        popped={poppedReply} 
                        setPopped={setPoppedReply} 
                        setActivities={setActivities} 
                        activities={activities} 
                    />
                    <ActivityFeed 
                        title="Notifications" 
                        setReplyActivity={setReplyActivity} 
                        setPopped={setPoppedReply} 
                        activities={activities} 
                    />
                </div>
            <DesktopSidebar user={user} />
        </article>
        );
        }
        ```

- This is what you should see after this step is done:
  ![week1_notifi-prrof-of-work](https://user-images.githubusercontent.com/123767474/220918691-51a6dcc0-92b3-43f0-837b-74f7c705d3f2.png)

6. Setting up DynamoDB Local and Postgres Container in docker-compose:
    - Modify the docker-compose.yaml file with the following
        ```sh
        services:
            dynamodb-local:
            # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
            # We needed to add user:root to get this working.
            user: root
            command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
            image: "amazon/dynamodb-local:latest"
            container_name: dynamodb-local
            ports:
                - "8000:8000"
            volumes:
                - "./docker/dynamodb:/home/dynamodblocal/data"
            working_dir: /home/dynamodblocal

            db:
                image: postgres:13-alpine
                restart: always
                environment:
                    - POSTGRES_USER=postgres
                    - POSTGRES_PASSWORD=password
                ports:
                    - '5432:5432'
                volumes: 
                    - db:/var/lib/postgresql/data

        volumes:
            db:
                driver: local
        ```

    - Then do docker-compose up. Make sure ports are unlocked!

7. Setting up DynamoDB:
    - Run the following to create a table:
        ```sh
        aws dynamodb create-table \
            --endpoint-url http://localhost:8000 \
            --table-name Music \
            --attribute-definitions \
                AttributeName=Artist,AttributeType=S \
                AttributeName=SongTitle,AttributeType=S \
            --key-schema AttributeName=Artist,KeyType=HASH          AttributeName=SongTitle,KeyType=RANGE \
            --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
            --table-class STANDARD       
        ```
    - Run the following to create an item:
        ```sh
        aws dynamodb put-item \
            --endpoint-url http://localhost:8000 \
            --table-name Music \
            --item \
                '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}}' \
            --return-consumed-capacity TOTAL  
        ```
    - To list the table, run:
        ```sh
        aws dynamodb list-tables --endpoint-url http://localhost:8000
        ```
    - To scan for records, run:
        ```sh
        aws dynamodb scan --table-name Music --query "Items" --endpoint-url http://localhost:8000
        ```

  ![week1-proof-work](https://user-images.githubusercontent.com/123767474/220918340-8cd1debd-e28c-46e7-b52e-519ec3681f03.png)

  

7. Setting up Postgres:

    - Modify the docker-compose.yaml, this step is done to install postgres client:
        ```sh
          - name: postgres
            init: |
                curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
                echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
                sudo apt update
                sudo apt install -y postgresql-client-13 libpq-dev
        ```
    - Run each of the command above in terminal for installation.

    - To run the client:
        ```sh
        psql - Uposstgres --host localhost

        #Enter password: password
        ```
    - To test you can run some commands, you should get some output:
      ![post-week2](https://user-images.githubusercontent.com/123767474/220917864-f9d89ee7-1680-48b0-bac6-46e01c22ed5e.png)

</details>

---


<details>
<summary>Extended homework</summary>
        For this week's homework challenge, I performed the health check on docker-compose file. I followed a simple guide i found online and implemented it.
 
   ```sh
      healthcheck:
         test: curl --fail -s http://localhost:4567 || exit 1
         interval: 30s
         timeout: 30s
         retries: 3
   ```
   
   
   ![w1_ext](https://user-images.githubusercontent.com/123767474/221367466-66a75acd-5b21-4edd-9fb1-c9a9122f6e48.png)

- Here's what each of the attributes mean:

  - test: This specifies the command to run to check the health of the container. In this example, the command is curl --fail -s http://localhost:4567 || exit 1, which tries to fetch the URL http://localhost:4567 using curl and exits with a non-zero status code if the URL is not reachable. This means that if the test command fails (i.e., exits with a non-zero status code), the container will be considered unhealthy.

  - interval: This specifies the interval between health checks. In this example, the interval is set to 30s, which means that the health check will be run every 30 seconds.

  - timeout: This specifies the maximum amount of time to wait for the test command to complete before considering it a failure. In this example, the timeout is set to 30s, which means that if the test command takes longer than 30 seconds to complete, the container will be considered unhealthy.

  - retries: This specifies the number of times to retry running the test command before considering the container unhealthy. In this case, the retries are set to 3, which means that if the test command fails three times in a row, the container will be considered unhealthy.

- By using a health check configuration like this in  Docker Compose file, we can ensure that our containerized application or service is always running and healthy, and that any issues are detected and resolved quickly.

- Next challenge I attempted was to learn how to install Docker on your local machine and get the same containers running outside of Gitpod. This was smooth and i was able to run my containers from my local machine in localhost but the docker compose up took near about 30 minutes to complete, so for now i have decided i have decided not to use my local with my 4gigs ram but i will be using it for testing and doing homework challenges without worrying about running out of gitpod credits.

- Next i wanted to try multi-build, eventhough i somewhat understood how it is done but i couldn't figure out how to implement in our project. Due to time constraint i have decided to implement at a later stage.
</details>

---
<details>
<summary>Resource used</summary>

1. Content:<br>
        - Official cloud project bootcamp playlist <br>
        - KodeKloud docker course
    
    _Note: All images used in the journal are made using figma._

</details>
