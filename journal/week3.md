<img width="1584" alt="W1-B" src="https://user-images.githubusercontent.com/123767474/224489595-4ae26e56-cc89-402e-94b5-be118180bac4.png">


# Week 3 — Decentralized Authentication

<details>

<summary>Tools of the week</summary>

- `Amazon Cognito`: It is a managed service provided by AWS that enables you to add user sign-up, sign-in, and access control to your web and mobile apps. One of the key features of Cognito is User Pools. <br> A user pool is a user directory in Cognito that allows you to create and manage a set of app users, including their sign-up, sign-in, and authentication details. It provides features such as user registration, authentication, and account recovery.

- `AWS Amplify`: It is a development platform offered by Amazon Web Services (AWS) that allows developers to quickly and easily create and deploy web and mobile applications. It provides a set of tools and services for building cloud-powered applications using popular frameworks like React, Angular, Vue, and more. Features include - Authentication, APIs, analytics and hosting.
 
</details>

---

<details>
<summary>Required homework</summary>

#### 1.  Create a new user pool in Amazon cognito:
    <br>

    - Configure sign-in experience:
      - Provider types: Cognito user pool.
      - Cognito user pool sign-in options: Email.
    - Configure security requirements:
      - Password policy mode: cruddur-user-poo
      - MFA enforcement: No MFA.
      - Self-service account recovery: Self-service account recovery.
      - Delivery method for user account recovery messages: Email only.
    - Configure sign-up experience:
      - Self-registration: Enable Self-registration
      - Cognito-assisted verification and confirmation: Enable it.
      - Attributes to verify: Send email message, verify email address.
      - Keep original attribute value active when an update is pending : Enabled
      - Additional required attributes: name, preferred_username
    - Configure message delivery:
      - Email provider: Send email with Cognito
    - Integrate your app:

      - User pool name: cruddur-userpool
      - App type: Public client
      - App client name: cruddur
      - Client secret: Don't generate a client secret. <br>

      Review and create!<br/>

#### 2.  Setting up cognito: <br/>

    - Install AWS amplify in `/frontend-react`:
      ```sh
      npm i aws-amplify --save
      ```
    - Go into `app.js`:
      ```js
      import { Amplify } from 'aws-amplify';
      Amplify.configure({
          "AWS_PROJECT_REGION": process.env.REACT_APP_AWS_PROJECT_REGION,
          "aws_cognito_region": process.env.REACT_APP_AWS_COGNITO_REGION,
          "aws_user_pools_id": process.env.REACT_APP_AWS_USER_POOLS_ID,
          "aws_user_pools_web_client_id": process.env.REACT_APP_CLIENT_ID,
          "oauth": {},
      Auth: {
          // We are not using an Identity Pool
          // identityPoolId: process.env.REACT_APP_IDENTITY_POOL_ID, // REQUIRED - Amazon Cognito Identity Pool ID
          region: process.env.REACT_APP_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
          userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
          userPoolWebClientId: process.env.REACT_APP_CLIENT_ID,   // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
      }
      });
      ```
    - Setup in `gitpod.yaml`:
      ```yaml
      REACT_APP_AWS_PROJECT_REGION: "$(AWS_DEFAULT_REGION)"
      REACT_APP_AWS_COGNITO_REGION: "$(AWS_DEFAULT_REGION)"
      REACT_APP_AWS_USER_POOLS_ID: "user_pool_id"
      REACT_APP_CLIENT_ID: "app_client_id"
      ```
    - Conditionally show components based on logged in or logged out: <br/>

      Go to `HomeFeedPage.js`

      ```js
      import { Auth } from "aws-amplify";

      //Replace check auth function

      // check if we are authenticated
      const checkAuth = async () => {
        Auth.currentAuthenticatedUser({
          // Optional, By default is false.
          // If set to true, this call will send a
          // request to Cognito to get the latest user data
          bypassCache: false,
        })
          .then((user) => {
            console.log("user", user);
            return Auth.currentAuthenticatedUser();
          })
          .then((cognito_user) => {
            setUser({
              display_name: cognito_user.attributes.name,
              handle: cognito_user.attributes.preferred_username,
            });
          })
          .catch((err) => console.log(err));
      };
      ```

    - Update `ProfileInfo.js`:

      ```js
      //TODO Authentication, :
      import { Auth } from "aws-amplify";

      //replace const signOut
      const signOut = async () => {
        try {
          await Auth.signOut({ global: true });
          window.location.href = "/";
        } catch (error) {
          console.log("error signing out: ", error);
        }
      };
      ```

#### 3.  Implementing Custom Signin Page:

    - Go to `src/pages/SigninPage.js`:

      ```js
      //TODO authentication
      //replace cookie with
      import { Auth } from "aws-amplify";

      //Replace onsubmit function
      const onsubmit = async (event) => {
        setErrors("");
        event.preventDefault();
        Auth.signIn(email, password)
          .then((user) => {
            console.log("user", user);
            localStorage.setItem(
              "access_token",
              user.signInUserSession.accessToken.jwtToken
            );
            window.location.href = "/";
          })
          .catch((error) => {
            if (error.code == "UserNotConfirmedException") {
              window.location.href = "/confirm";
            }
            setErrors("")(error.message);
          });
        return false;
      };
      ```

    - Go into cognito console and create user inside the the usergroup:
      - Alias attributes used to sign in: Enable Email
      - User name: spacecadet
      - Email address: asthaghosh.it @gmail.com
      - Password: Test#1234

    Note: we were unable to confirm the identity if the user through console we will use cli instead:

    ```sh
    aws cognito-idp admin-set-user-password --username spacecadet --password Test#1234 --user-pool-id [tbd] --permanent
    ```

#### 4.  Implementing Custom Signup Page:

    - Go to `src/pages/SignupPage.js`:

    ```js
      //TODO authentication
      //replace cookie with
      import { Auth } from "aws-amplify";

      //Replace onsubmit function

      const onsubmit = async (event) => {
        event.preventDefault();
        setErrors("");
        try {
          const { user } = await Auth.signUp({
            username: email,
            password: password,
            attributes: {
              name: name,
              email: email,
              preferred_username: username,
            },
            autoSignIn: {
              // optional - enables auto sign in after user is confirmed
              enabled: true,
            },
          });
          console.log(user);
          window.location.href = `/confirm?email=${email}`;
        } catch (error) {
          console.log(error);
          setErrors(error.message);
        }
        return false;
      };
      ```

#### 5.  Implementing Custom Conformation Page:

    - Go to `src/pages/ConformationPage.js`:

    ```js
      //TODO authentication
      //replace cookie with
      import { Auth } from "aws-amplify";

      //Replace the resend code
      const resend_code = async (event) => {
        setErrors("");
        try {
          await Auth.resendSignUp(email);
          console.log("code resent successfully");
          setCodeSent(true);
        } catch (err) {
          // does not return a code
          // does cognito always return english
          // for this to be an okay match?
          console.log(err);
          if (err.message == "Username cannot be empty") {
            setcognitoErrors(
              "You need to provide an email in order to send Resend Activation Code"
            );
          } else if (err.message == "Username/client id combination not found.") {
            setcognitoErrors("Email is invalid or cannot be found.");
          }
        }
      };

      //replace the onsubmit
      const onsubmit = async (event) => {
        event.preventDefault();
        setErrors("");
        try {
          await Auth.confirmSignUp(email, code);
          window.location.href = "/";
        } catch (error) {
          setErrors(error.message);
        }
        return false;
      };
    ```

#### 6.  Implementing Custom Recovery Page:

    - Go to `src/pages/RecoveryPage.js`:

    ```js
    import { Auth } from "aws-amplify";

    const onsubmit_send_code = async (event) => {
      event.preventDefault();
      setErrors("");
      Auth.forgotPassword(username)
        .then((data) => setFormState("confirm_code"))
        .catch((err) => setErrors(err.message));
      return false;
    };
    const onsubmit_confirm_code = async (event) => {
      event.preventDefault();
      setErrors("");
      if (password == passwordAgain) {
        Auth.forgotPasswordSubmit(username, code, password)
          .then((data) => setFormState("success"))
          .catch((err) => setErrors(err.message));
      } else {
        setCognitoErrors("Passwords do not match");
      }
      return false;
    };
    ```

#### 7.  Congito JWT Server side Verification:

    - Go to `src/pages/HomeFeedPage.js`:
      ```js
      // paste the code below above method: "GET" });
      headers: {
        Authorization: `Bearer ${localStorage.getItem("access_token")}`
      },
      ```
    - Get the header request in backend-flask, got to `app.py`:

      ```python
      # import library
      import sys

      #Modify cors
      cors = CORS(
        app,
        resources={r"/api/*": {"origins": origins}},
        headers=['Content-Type', 'Authorization'],
        expose_headers='Authorization',
        methods="OPTIONS,GET,HEAD,POST"
      )
      #To check if the token is being passed along
      #Add in def data_home():
      app.logger.debug('AUTH HEADER');
      app.logger.debug(AUTH HEADER);
      app.logger.debug(
        request.headers.get('Authorization')
      )
      #data = HomeActivities.run() #Logger = LOGGER
      #return data, 200
      #Once checked remove the part
      ```

    - Make a new folder and file as such `backend-flask/lib/cognito_jwt_token.py`:

      ```py
        import time
        import requests
        from jose import jwk, jwt
        from jose.exceptions import JOSEError
        from jose.utils import base64url_decode

        class FlaskAWSCognitoError(Exception):
            pass


        class TokenVerifyError(Exception):
          pass
          def extract_access_token(request_headers):
                access_token = None
                auth_header = request_headers.get("Authorization")
                if auth_header and " " in auth_header:
                    _, access_token = auth_header.split()
                return access_token


        class CognitoJwtToken:
            def __init__(self, user_pool_id, user_pool_client_id, region, request_client=None):
                self.region = region
                if not self.region:
                    raise FlaskAWSCognitoError("No AWS region provided")
                self.user_pool_id = user_pool_id
                self.user_pool_client_id = user_pool_client_id
                self.claims = None
                if not request_client:
                    self.request_client = requests.get
                else:
                    self.request_client = request_client
                self._load_jwk_keys()

            def _load_jwk_keys(self):
                keys_url = f"https://cognito-idp.{self.region}.amazonaws.com/{self.user_pool_id}/.well-known/jwks.json"
                try:
                    response = self.request_client(keys_url)
                    self.jwk_keys = response.json()["keys"]
                except requests.exceptions.RequestException as e:
                    raise FlaskAWSCognitoError(str(e)) from e

            @staticmethod
            def _extract_headers(token):
                try:
                    headers = jwt.get_unverified_headers(token)
                    return headers
                except JOSEError as e:
                    raise TokenVerifyError(str(e)) from e

            def _find_pkey(self, headers):
                kid = headers["kid"]
                # search for the kid in the downloaded public keys
                key_index = -1
                for i in range(len(self.jwk_keys)):
                    if kid == self.jwk_keys[i]["kid"]:
                        key_index = i
                        break
                if key_index == -1:
                    raise TokenVerifyError("Public key not found in jwks.json")
                return self.jwk_keys[key_index]

            @staticmethod
            def _verify_signature(token, pkey_data):
                try:
                    # construct the public key
                    public_key = jwk.construct(pkey_data)
                except JOSEError as e:
                    raise TokenVerifyError(str(e)) from e
                # get the last two sections of the token,
                # message and signature (encoded in base64)
                message, encoded_signature = str(token).rsplit(".", 1)
                # decode the signature
                decoded_signature = base64url_decode(encoded_signature.encode("utf-8"))
                # verify the signature
                if not public_key.verify(message.encode("utf8"), decoded_signature):
                    raise TokenVerifyError("Signature verification failed")

            @staticmethod
            def _extract_claims(token):
                try:
                    claims = jwt.get_unverified_claims(token)
                    return claims
                except JOSEError as e:
                    raise TokenVerifyError(str(e)) from e

            @staticmethod
            def _check_expiration(claims, current_time):
                if not current_time:
                    current_time = time.time()
                if current_time > claims["exp"]:
                    raise TokenVerifyError("Token is expired")  # probably another exception

            def _check_audience(self, claims):
                # and the Audience  (use claims['client_id'] if verifying an access token)
                audience = claims["aud"] if "aud" in claims else claims["client_id"]
                if audience != self.user_pool_client_id:
                    raise TokenVerifyError("Token was not issued for this audience")

            def verify(self, token, current_time=None):
                """ https://github.com/awslabs/aws-support-tools/blob/master/Cognito/decode-verify-jwt/decode-verify-jwt.py """
                if not token:
                    raise TokenVerifyError("No token provided")

                headers = self._extract_headers(token)
                pkey_data = self._find_pkey(headers)
                self._verify_signature(token, pkey_data)

                claims = self._extract_claims(token)
                self._check_expiration(claims, current_time)
                self._check_audience(claims)

                self.claims = claims
                return claims
      ```

    - Add envars in `docker-compose.yml`:
      ```yml
      AWS_COGNITO_USER_POOL_ID = "eu-west-1_XXX"
      AWS_COGNITO_USER_POOL_CLIENT_ID= "YYY"
      ```
    - Go to `app.py`, paste:

      ```py
      from lib.cognito_jwt_token import CognitoJwtToken, extract_access_token, TokenVerifyError

      #app = Flask(__name__)

      cognito_jwt_token =  CognitoJwtToken(
        user_pool_id=os.getenv("AWS_COGNITO_USER_POOL_ID"),
        user_pool_client_id=os.getenv("AWS_COGNITO_USER_POOL_CLIENT_ID"),
        region=os.getenv("AWS_DRFAULT_REGION"),
      )

      #def data_home():
        access_token = extract_access_token(request.headers)
        try:
            claims = cognito_jwt_token.verify(access_token)
            #authenticated request
            app.logger.debug("authenticated")
            app.logger.debug('claims')
            app.logger.debug(claims['username'])
            data = HomeActivities.run(cognito_user_id=claims['username'])
        except TokenVerifyError as e:
            #unauthenticated request
          app.logger.debug(e)
          app.logger.debug("unauthenticated")
      ```

    - Go to `services/home_activities.py`:

      ```py

          def run(cognito_user_id=None):

          #'replies': []
          #}
          #]
            if cognito_user_id != None:
              extra_crud = {
                'uuid': '248959df-3079-4947-b847-9e0892d1bab4',
                'handle':  'Wabisabi',
                'message': 'Finding beauty in the imperfection and impermanence of things.',
                'created_at': (now - timedelta(hours=1)).isoformat(),
                'expires_at': (now + timedelta(hours=12)).isoformat(),
                'likes': 90,
                'replies': []
              }

              results.insert(0,extra_crud)

      #span.set_attribute("app.result_length", len(results))
      #return results
      ```

    - Go to `src/components/Profileinfo.js`:
      ```js
      //try {
      //await Auth.signOut({ global: true });
      //window.location.href = "/"
      localStorage.removeItem("access_token");
      //} catch (error) {
      //console.log('error signing out: ', error);
      //}
 
      ```
      After login we would see something like this: 
      ![week_3](https://user-images.githubusercontent.com/123767474/224489434-3dc023dd-4dd9-49a0-a09a-756ef93b7904.png)
</details>

---
