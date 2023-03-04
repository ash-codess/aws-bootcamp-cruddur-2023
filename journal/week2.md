<img width="1584" alt="W1-B(2)" src="https://user-images.githubusercontent.com/123767474/222810511-46d730f6-2bb2-4631-a328-244ea4958d10.png">

# Week 2 — Distributed Tracing


![GitHub last commit (branch)](https://img.shields.io/github/last-commit/ash-codess/aws-bootcamp-cruddur-2023/main)

<details>
<summary>
Youtube live takeaway
</summary>
- This week we started of with distributed tracing. During live we instrumented our backend-flask with honeycomb which we will bw using for observability in coming weeks as we add more services and functionality.
   
- Monitoring Vs Observability: <br> 
    - Monitoring refers to the process of collecting data from various sources (such as logs, metrics, and traces) to understand the current state of a system and detect problems or anomalies. Monitoring typically involves setting up thresholds, alerts, and dashboards to visualize the data and identify issues that require attention.
    
    - Observability, on the other hand, is a more holistic approach to understanding a system's behavior and performance. Observability involves designing a system so that its internal state and external behavior can be inferred from its outputs. Observability tools typically provide insights into the system's internal workings, including its dependencies, performance bottlenecks, and error conditions, to help operators and developers quickly diagnose and troubleshoot issues.

- Few important terminology:<br>
    - Telemetry: Data that the software emits to tell it's team what's going on inside.
    - Instrumentation: Code that emits telemetry.
        - auto-instrumentation: Code we didn't write, that emits telemetry.
    - Open-telemetry: Initialization of code to generate telemetry.

        <img width="592" alt="honeycomb" src="https://user-images.githubusercontent.com/123767474/222814200-bc85ceed-d7c9-45ae-989a-90b398199677.png">
    
- Tools we will be using this week:
    - `Honeycomb`: Honeycomb.io is a cloud-based observability platform that provides developers and operators with deep insights into their systems and applications. The platform enables teams to collect, analyze, and visualize high-dimensional data in real-time, making it easier to diagnose and troubleshoot issues, and improve the overall performance of their applications.<br/>
    - `AWS X-Ray`: It is a distributed tracing system that enables developers to analyze and debug applications running on AWS. It allows you to track requests as they flow through your application, identifying performance bottlenecks and errors in real-time.  The system provides detailed information about each request, including request origin, timings, service calls, and exceptions.
    - `Amazon CloudWatch Logs`: It is a log management service provided by AWS. It allows you to monitor, store, and access log files from your applications, operating systems, and other AWS services. CloudWatch Logs can be used to monitor logs in real-time, set alarms on specific log events, and analyze log data using CloudWatch Insights.
    - `Rollbar`: It is a powerful tool for monitoring errors and exceptions in our applications. It provides a centralized view of errors across your application stack, streamlines your development workflow, and helps you identify and fix issues quickly.
</details> 

---


<details>
<summary>
Required Homework
</summary>
<br>

## 1. Instrumenting with Honeycomb

<br>

- Create a new environment in [honeycomb](https://www.honeycomb.io/). (By default a "test" environment already exists)

- Setup API key and service name using the api key provided. Service name is custom but it should be relevant in our case its: backend-flask
  ```
  export HONEYCOMB_API_KEY="[api-key-here]"
  export HONEYCOMB_SERVICE_NAME="backend-flask"
  gp env HONEYCOMB_API_KEY="[api-key-here]"
  gp env HONEYCOMB_SERVICE_NAME="backend-flask"
  ```
- Add envars to `docker-compose.yml`:
  ```
  OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
  OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
  OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
  ```
- Setup the `requirements.txt` in backend-flask:
  ```
  opentelemetry-api
  opentelemetry-sdk
  opentelemetry-exporter-otlp-proto-http
  opentelemetry-instrumentation-flask
  opentelemetry-instrumentation-requests
  ```
- Run pip command to install the dependencies:
  ```
  pip install -r requirements.txt
  ```
- Import opentelemetry libraries in `app.py`

  ```
  #Honeycomb(importing libraries)

  from opentelemetry import trace
  from opentelemetry.instrumentation.flask import FlaskInstrumentor
  from opentelemetry.instrumentation.requests import RequestsInstrumentor
  from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
  from opentelemetry.sdk.trace import TracerProvider
  from opentelemetry.sdk.trace.export import BatchSpanProcessor
  from opentelemetry.sdk.trace.export import ConsoleSpanExporter, SimpleSpanProcessor
  ```

- Start the instrumenting process by adding to `app.py`

  ```
  # Initialize tracing and an exporter that can send data to Honeycomb

  provider = TracerProvider()
  processor = BatchSpanProcessor(OTLPSpanExporter())
  provider.add_span_processor(processor)
  ```

  ```
  trace.set_tracer_provider(provider)
  tracer = trace.get_tracer(__name__)

  # Initialize automatic instrumentation with Flask
  # insert this after app = Flask(__name__)
  FlaskInstrumentor().instrument_app(app)
  RequestsInstrumentor().instrument()
  ```

- Add a new span for testing:
  ```sh
  #Show this in the logs within the backend-flask app (STDOUT)
  simple_processor = SimpleSpanProcessor(ConsoleSpanExporter())
  provider.add_span_processor(simple_processor)
  ```
- Adding tracer in `homeActivities.py`

  ```sh
      from opentelemetry import trace

      tracer = trace.get_tracer("home.activities")

      #def run():
          with tracer.start_as_current_span("home-activites-mock-data"):
              span = trace.get_current_span()
              now = datetime.now(timezone.utc).astimezone()
              span.set_attribute("app.now", now.isoformat())

              #----------------code-----------

              span.set_attribute("app.result_length", len(results))
      return results
  ```

- Spin up docker and go to backend endpoint and hit refresh couple of times to receive traces on honeycomb dashboard.

- Creating segments and sub-segments to receive traces on honeycomb dashboard:

  ![w2-honeycomb](https://user-images.githubusercontent.com/123767474/222811014-9e9f5a44-efb6-4836-8f79-e73595835ed9.png)

- Troubleshooting tips:

  - Check if the api key is correct [here](https://honeycomb-whoami.glitch.me/)
  - Hit refresh couple of times to receive traces
  - Commit changes and restart gitpod workspace
  - Check envars is setup in container correctly using attach shell.

<br>

## 2. Instrumenting with xray

<br>
<img width="533" alt="xray" src="https://user-images.githubusercontent.com/123767474/222814306-e016e992-82fe-4b65-a4b7-95d172588cc9.png">

- Add the xray-sdk `requirements.txt`:
  ```sh
  aws-xray-sdk
  ```
- Run pip command to install the dependencies:
  ```sh
  pip install -r requirements.txt
  ```
- Add recorder and middleware to `app.py`:

  ```sh
  from aws_xray_sdk.core import xray_recorder
  from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

  xray_url = os.getenv("AWS_XRAY_URL")
  xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)

  #app = FLASK(__name__)

  XRayMiddleware(app, xray_recorder)
  ```

- Make a new json file for setting up xray sampling rule at `aws/json/xray.json`:
  ```sh
  {
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
      }
  }
  ```
- Create xray group, by running:

  ```sh
  aws xray create-group \
      --group-name "Cruddur" \
      --filter-expression "service(\"backend-flask\") "
  ```
   
 ![week-2-xray-op](https://user-images.githubusercontent.com/123767474/222815325-c19eb289-505b-4142-b6f7-8a71165b414a.png)



- Create a sampling rule:
  ```sh
  aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
  ```
- Add daemon service to `docker-compose.yml` file to run in container:
  ```sh
  xray-daemon:
      image: "amazon/aws-xray-daemon"
  environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
  command:
      - "xray -o -b xray-daemon:2000"
  ports:
      - 2000:2000/udp
  ```
- Adding envars in `docker-compose.yml` for xray:
  ```sh
  AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
  AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
  ```
- Modify `app.py` to add xray_recorder.capture:

  ```sh
  @app.route("/api/activities/home", methods=['GET'])
  @xray_recorder.capture('activities_home')
  def data_home():
      data = HomeActivities.run() #Logger = LOGGER
      return data, 200

  @app.route("/api/activities/@<string:handle>", methods=['GET'])
  @xray_recorder.capture('activities_users')
  def data_handle(handle):
      model = UserActivities.run(handle)
      if model['errors'] is not None:
          return model['errors'], 422
      else:
          return model['data'], 200
  ```

- Create a segments/subsegment in `userActivities.py`:

  ```sh
  def run(user_handle):
    try:
        model = {
            'errors': None,
            'data': None
            }

        subsegment = xray_recorder.begin_subsegment('mock-data')

        dict = {
            "now": now.isoformat(),
            "results-size": len(model['data'])
        }
            subsegment.put_metadata('key', dict, 'namespace')
            xray_recorder.end_subsegment()
    finally:
        xray_recorder.end_subsegment()

    return model
  ```

- We can see segments being sent when we hit refresh at backend endpoint. The url is modified by adding a user at the end
  ![w2-sub-seg0terminal](https://user-images.githubusercontent.com/123767474/222811119-1c88c7b6-6f82-408f-8657-f7aacc65bd5b.png)
  Finally check the cloudwatch console for traces of subsegments:
  ![w2_cw-sub-segment](https://user-images.githubusercontent.com/123767474/222811138-fe6ac3d1-a5ee-404e-96b1-154da652c349.png)
  <br>


## 3. Setting up Cloudwatch


<br>

- Add watchtower to `requirements.txt` then do pip install:

  ```
  watchtower

  pip install -r requirements.txt
  ```

- Import library to `app.py`:
  ```sh
  import watchtower
  import logging
  from time import strftime
  ```
- Configuring Logger to Use CloudWatch in `app.py`
  ```sh
  LOGGER = logging.getLogger(__name__)
  LOGGER.setLevel(logging.DEBUG)
  console_handler = logging.StreamHandler()
  cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
  LOGGER.addHandler(console_handler)
  LOGGER.addHandler(cw_handler)
  LOGGER.info("test log")
  ```
- For error logging:
  ```sh
  @app.after_request
  def after_request(response):
      timestamp = strftime('[%Y-%b-%d %H:%M]')
      LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
      return response
  ```
- To start instrumenting, add these to `home_activities.py`:

  ```sh
  import logging

  def run(Logger):
  Logger.info("HomeActivities")
  ```

- Modify the homeActivities with logger in `app.py`:
  ```sh
  @app.route("/api/activities/home", methods=['GET'])
  def data_home():
      data = HomeActivities.run(Logger = LOGGER)
      return data, 200
  ```
- Setup envars for watchtower in `docker-compose.yml`:

  ```sh
  #insert below xray-daemon

  AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
  AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
  AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
  ```

- Spin up docker then go to backend endpoint and refresh it couple of times. Finally check the cloudwatch log console.
  It should appear like this, you should see logs streaming in!
  ![w2-logs](https://user-images.githubusercontent.com/123767474/222811901-89f8266d-06e0-48d9-812b-96b5042beda3.png)

- To keep the cost low for cloudwatch logs, turn it off for now by making modifications in code:

  ```sh
  #in app.py
  @app.route("/api/activities/home", methods=['GET'])
  def data_home():
      data = HomeActivities.run()
      return data, 20

  #in home_activities.py
  def run():
  #Logger.info("HomeActivities")
  ```

## 4. Setting up Rollbar

- Setup a new project in rollbar
- Add blinker and rollbar to `requirements.txt`:
  ```sh
  blinker
  rollbar
  ```
- Run pip command to install the dependencies:

  ```sh
  pip install -r requirements.txt
  ```

- Add rollbar access token to gitpod, run the following command:

  ```sh
  export rollbar_accessToken="token_goes_here"
  gp env rollbar_accessToken="token_goes_here"

  env | grep rollbar_accessToken #for checking
  ```

- Add access token to docker-compose.yml:

  ```sh
  ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
  ```

- Now as first step of instrumentation, add libraries to `app.py`:
  ```sh
  import os
  import rollbar
  import rollbar.contrib.flask
  from flask import got_request_exception
  ```
- Initialize rollbar in `app.py`:

  ```sh
  rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
  @app.before_first_request
  def init_rollbar():
      """init rollbar module"""
      rollbar.init(
          # access token
          rollbar_access_token,
          # environment name
          'production',
          # server root directory, makes tracebacks prettier
          root=os.path.dirname(os.path.realpath(__file__)),
          # flask already sets up logging
          allow_logging_basic_config=False)

      # send exceptions from `app` to rollbar, using flask's signal system.
      got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
  ```

- Make a new endpoint to test rollbar:
  ```sh
  @app.route('/rollbar/test')
  def rollbar_test():
      rollbar.report_message('Hello World!', 'warning')
      return "Hello World!"
  ```
- Spin up docker and go to backend point then modify the url with `/rollbar/test` <br>
  We introduce error in the code to test bug reporting in rollbar
  ![w2-roll](https://user-images.githubusercontent.com/123767474/222811377-ab99633e-6857-4baf-b678-23ad3828cd27.png)

</details>

---

<details>
<summary>
Extended Homework
</summary>
pending work
</details>

---

<details>
<summary>
Resources used
</summary>

1. Content:<br>
        - Official cloud project bootcamp playlist <br>
        - Observability course 
    
    _Note: All images used in the journal are made using figma._

</details>
