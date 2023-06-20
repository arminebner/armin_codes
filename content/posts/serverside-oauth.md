---
title: "Serverside Oauth2.0 Authentication"
date: 2023-06-20T16:47:14+02:00
draft: false
tags:
  ["typescript", "node.js", "nest.js", "google", "oauth", "authentication", "learning", "fullstack"]
cover:
  image: "https://arminebner.codes/heroimage-authentication.png"
  alt: "an abstract dalle created image that is supoposed to be a hero i,age for a blog article about authentication"
  caption: "<text>"
---

# Google OAuth2.0 Server-Side Authentication

## Background information

### What is authentication

Authentication is the process of verifying the identity of a user or entity. It ensures that the claimed identity of a user or entity is valid and legitimate. Authentication typically involves the presentation of credentials, such as a username and password, biometric data, smart cards, or digital certificates.  
The goal of authentication is to establish trust and ensure that only authorized individuals or entities gain access to a system or resource.

On the other hand, authorization happens after authentication. Authorization establishes what activities or resources a person or entity is permitted to access after successful authentication and identity establishment.  
According to the job, group membership, or particular access policies of the authenticated user, authorization entails granting or rejecting permissions, rights, or privileges.  
It limits unwanted access to sensitive data or functionality and specifies the range of permitted operations.

In summary, authentication is the process of verifying one's identity, while authorization determines what a user can do or which resourcces he can access after their identity is authenticated. Authentication establishes trust, while authorization controls and manages access permissions.

### Why do it on the server

Flexibility and Scalability: You have more control and flexibility in designing and administering access restrictions when authorization is managed on the server side.  
This makes it simple to modify access policies, roles, or permissions without having to make any modifications to the client-side application.  
By centralizing authorization logic and making access control easier to maintain as your application expands and it helps with scalability.

Consistency: When multiple client apps may access the same backend resources, server-side permission helps preserve consistency across all of these applications.  
No matter the client being used, you can guarantee that access control rules are applied consistently by implementing authorization on the server side.

Protection against client-side manipulation: Client-side authorization can be easily manipulated or bypassed by malicious users. By performing authorization checks on the server side, you reduce the risk of unauthorized access or tampering with access controls.

## Example using NestJS and PassportJS Google strategy

### The setup

#### Backend

We have a controller that is responsible for all authentication related requests coming from the client that could look like this:

```js
// authentication.controller.ts
@Controller('authentication')
export class AuthController {
    constructor(
        private config: ConfigService,
    ) {}
}
```

For our desired auth flow, we need to provide two endpoints in the **auth.controller.ts** file

```js
// authentication.controller.ts
@Controller('authentication')
export class AuthController {
    constructor(
        private config: ConfigService,
    ) {}

    @Get('google')
    @UseGuards(AuthGuard('google'))
    googleLogin() {
        // initiates the Google OAuth2 login flow
    }
}
```

Let's have a closer look the first endpoint for now:
If we hit **/auth/google**, the Google OAuth2 login flow courtesy of PassportJS gets triggered.
Before you call that, you have to configure it.  
If you choose to use the build in google strategy (PassportJS uses a lot of different strategies that zou can configure. For me information see [PassportJD Docs](https://www.passportjs.org/docs/)) then create a file called **google.strategy.ts**.
In there we extend the PassportStrategy and we have a constructor and/or a super-constructor plus a validate method.

```js
// google.strategy.ts
@Injectable()
export class GoogleStrategy extends PassportStrategy(Strategy, 'google') {
    constructor(
      private configService: ConfigService,
      private authService: AuthService,
    ) {
        super({
            clientID: configService.get('CLIENT_ID'),
            clientSecret: configService.get('CLIENT_SECRET'),
            callbackURL: configService.get('CALLBACK_URL'),
            scope: ['email', 'profile'],
        });
    }

    async validate(
        accessToken: string,
        refreshToken: string,
        profile: any,
        done: VerifyCallback,
    ) {
        {
            try {
                const user = profile._json;
                user.jwt = await this.authService.validateOAuthLogin(
                    profile._json,
                    Provider.GOOGLE,
                );
                done(null, user);
            } catch (err) {
               //console.log(err)
                done(err, false);
            }
        }
    }
}
```

For the constructor you need to provide variables for clientID, clientSecret, callbackURL and scope.
The value of these variables are set in the google cloud console while you are configuring your OAuth Project.  
What you want to achieve in the validate method is up to you. We are going for saving the user in our databank and returning a corresponding JSON webtoken that we then can use later on.

Remember: This all happens, when we call the **/auth/google** endpoint. We configure the strategy and what we want out of it. The rest is handled by PassportJS.

If the validate method has finished, the **CALLBACK_URL** we specified in the google cloud console settings comes into play and for this one, we need a second endpoint in our **auth.controller.ts** file which in our case responds to **/auth/google/callback**

```js
// authentication.controller.ts
@Controller('authentication')
export class AuthController {
    constructor(
        private config: ConfigService,
    ) {}

    @Get('google')
    @UseGuards(AuthGuard('google'))
    googleLogin() {
        // initiates the Google OAuth2 login flow
    }   
   
@Get('google/callback')
@UseGuards(AuthGuard('google'))
    googleLoginCallback(@Req() req, @Res() res) {
        // handles the Google OAuth2 callback
        const jwt: string = req.user.jwt;
        const encodedToken = encodeURIComponent(jwt)
        const redirectUrl = new URL(this.config.get('CALLBACK_FRONTEND'));
        if (jwt && redirectUrl) {
            res.redirect(redirectUrl.toString() + encodedToken);
        } else {
            res.redirect('../failure');
        }
    }
}
```

If this endpoint gets called we extract the JWT from the req.user object and send it via redirect as URL parameter to our frontend.
**Note:** This endpoint gets called in response to starting the oauth flow with PassportJS. We don not call this endpoint ourselves.

Again: What happens in this block of code is your decision based on what you need for the project is. You could just send the JWT back in JSON and then safe it in global state in your frontend.
And you would be done here.  
You have successfully let your user login via a google account and provided a JWT to further check authentification and / or authorization status of this user.

But we want to add another step to the process: Letting the user specify a short-form of his name consisting of three uppercase letters. Example: If your name is John Doe, you would be required to choose JDO.  
Therefore we need another controller, which we call **user.controller.ts**.
In there, we provide an endpoint that we can reach under **/users/register**.
It requires a valid JWT and uses the user data from the decoded JWT (we do this in the frontend in the next section) and the user input for his short form or "kuerzel".  
In the userService we look for an existing user and override the placeholder short form we created during the google oauth authentification process with the users input.
After that we return a new JWT and send it back to the frontend.

```js
// user.controller.ts
@Controller('users')
export class UsersController {
    constructor(private usersService: UsersService) {}

@Put('registerUser')
@UseGuards(AuthGuard('jwt'))
    async register(@Req() request: any): Promise<any> {
        const authorization = request.headers.authorization;
        let jwt;
        if (authorization) {
            jwt = authorization.replace('Bearer ', '');
        }
        const userData = request.body;
        return await this.usersService.register(userData, jwt);
    }
}
```

```js
// user.service.ts
@Injectable()
export class UsersService {
    constructor(
        @InjectModel('Users') private readonly users,
        private readonly config: ConfigService,
    ) {}

    async register(userData: any, jwt): Promise<any> {
        const user = await this.users.findOne({ mail: userData.email }).exec();
        user.short = userData.short;
        const expireTime = parseInt(this.config.get('JWT_EXPIRES_IN'), 10);
        return await this.users
            .findOneAndUpdate({ _id: user._id }, user)
            .then(data => {
                const payload = {
                    data: decode(jwt).data,
                };
                payload.data.short = user.short;
                const _return = sign(
                    payload,
                    this.config.get('JWT_SECRET_KEY'),
                    { expiresIn: expireTime },
                );
                return { jwt: _return };
            })
            .catch(reason => {
                return reason.code;
            });
    }
}
```

#### Frontend

So after hitting the "login with google" button in the frontend we get loggedin using our google account. After that we get redirected to https://frontend.com/sign/success/generated.JWT.from.backend.

Would you have decided to just send the JWT via JSON from your backend then you would just have to save the token to make it accessible from within your application. In this case, you would be done now.

Since we added an extra step though, we now need to make a distinction: Is the user a first time visitor? Then he needs to be able to put in his short from via a text input field and then hit the register button. After that he should be redirected to the restricted area of the website.  
If he's already registered we want to redirect him directly to the secured user-only area. To handle the dynamic routing and to access the token from the URL we create a dynamic route in our vue-router:

```js
    {
      path: '/sign/success/:token',
      name: 'registration',
      component: RegistrationScreen,
    },
```

Using VueJS in version 3 with script setup, the Logic in our RegistrationScreen.vue component looks like this:

```js
// RegistrationScreen.vue
const route = useRoute();
const { params } = route;
const token = params.token as string;
const decodedToken = jwtDecode<MyJwtPayload>(token);

const short = ref('');

onMounted(() => {
  const regex = /onHold(\d+)/g;
  if (!regex.test(decodedToken.data.short)) {
    setJwt(token);
    router.push('/home');
  }
});

const isValidInput = (input: string) => {
  const regex = /^[A-Z]{3}$/;
  return regex.test(input);
};

const setShort = (payload: string) => {
  short.value = payload;
};

const register = async (e: Event) => {
  e.preventDefault();

  const data = {
    email: decodedToken.data.email,
    short: short.value,
  };

  if (isValidInput(short.value)) {
    try {
      const result = await axios.put(
        'https://www.backend.com/users/registerUser',
        data,
        {
          headers: {
            Authorization: `Bearer ${token}`,
          },
        }
      );
      setJwt(result.data.jwt);
      // show success message
      router.push('/home');
    } catch (error) {
      console.error(error);
    }
  } else {
    // show error message
    console.log(short.value);
  }
};

```

When the component gets mounted we decode the token and check of which type the value of "short" has. Is it a placeholder-value then we show the input field and the registration button.  
After the user types his short-form and registers with it, we set the new token we get from calling the **users/register** endpoint into global state and redirect to the home page.

Is it already the user entered short-form, for example JDO for John Doe, then we directly set the token in our global state and redirect to the home page.

That's it for today - google auth2.0 authentification handled by our backend.

---

<span style="font-size:4px">Picture courtesy of [Dalle-2](https://openai.com/product/dall-e-2)</span>
