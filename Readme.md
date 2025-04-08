<pre> ```js 🔐 Secure REST APIs in Hapi.js – Complete Guide with Code
👨‍💻 Stack Used: Hapi.js, JWT, Joi, Helmet, Rate Limiting
🛡️ Goal: Build a secure API that prevents common attacks like XSS, brute force, and unauthorized access. ``` </pre>

<pre> ```js 1️⃣ Initialize a Basic Hapi.js Server ``` </pre>

<div style="background:#1e1e1e; color:#fff; padding:15px; border-radius:8px; font-family:monospace;">
  <pre><code>
npm init -y
npm install @hapi/hapi @hapi/joi hapi-auth-jwt2 jsonwebtoken hapi-rate-limit hapi-helmet dotenv
  </code></pre>
</div>

<div style="background:#1e1e1e; color:#fff; padding:15px; border-radius:8px; font-family:monospace;">
  <pre><code>
    // server.js
    require('dotenv').config();
    const Hapi = require('@hapi/hapi');

    const init = async () => {
    const server = Hapi.server({
        port: 4000,
        host: 'localhost'
    });

    await server.start();
    console.log('🚀 Server running on %s', server.info.uri);
    };

    init();
  </code></pre>
</div>

<pre> ```js 2️⃣ Add JWT Authentication ``` </pre>
(```a. Create a token generator (for demo):```)

<div style="background:#1e1e1e; color:#fff; padding:15px; border-radius:8px; font-family:monospace;">
  <pre><code>
    // utils/token.js
    const jwt = require('jsonwebtoken');

    const generateToken = (user) => {
    return jwt.sign(user, process.env.JWT_SECRET, { algorithm: 'HS256', expiresIn: '1h' });
    };

    module.exports = { generateToken };
  </code></pre>
</div>

(```b. Register JWT auth strategy:```)

<div style="background:#1e1e1e; color:#fff; padding:15px; border-radius:8px; font-family:monospace;">
  <pre><code>
    const Jwt = require('hapi-auth-jwt2');

    const validateUser = async (decoded, request, h) => {
    // Custom logic to validate user from DB
    if (decoded && decoded.userId) return { isValid: true };
    return { isValid: false };
    };

    await server.register(Jwt);

    server.auth.strategy('jwt', 'jwt', {
    key: process.env.JWT_SECRET,
    validate: validateUser,
    verifyOptions: { algorithms: ['HS256'] }
    });

    server.auth.default('jwt');
  </code></pre>
</div>


<pre> ```js 3️⃣ Secure Routes with JWT ``` </pre>

<div style="background:#1e1e1e; color:#fff; padding:15px; border-radius:8px; font-family:monospace;">
  <pre><code>
    server.route({
    method: 'GET',
    path: '/secure-data',
    handler: (request, h) => {
            return { message: 'This is a protected route', user: request.auth.credentials };
        }
    });
  </code></pre>
</div>


<pre> ```js 4️⃣ Validate Payloads with Joi (Prevents Injection & XSS) ``` </pre>

<div style="background:#1e1e1e; color:#fff; padding:15px; border-radius:8px; font-family:monospace;">
  <pre><code>
    const Joi = require('@hapi/joi');

    server.route({
    method: 'POST',
    path: '/login',
    options: {
        auth: false,
        validate: {
        payload: Joi.object({
            email: Joi.string().email().required(),
            password: Joi.string().min(6).required()
        }),
            failAction: (request, h, err) => {
                throw err;
            }
        }
    },
    handler: (request, h) => {
            const { email } = request.payload;
            const token = generateToken({ userId: 123, email });
            return { token };
        }
    });

  </code></pre>
</div>

<pre> ```js 5️⃣ Add Security Headers with Helmet ``` </pre>

<div style="background:#1e1e1e; color:#fff; padding:15px; border-radius:8px; font-family:monospace;">
  <pre><code>
    const helmet = require('hapi-helmet');
    await server.register(helmet);
  </code></pre>
</div>

<pre> ```js Helmet helps set headers like:

        * X-XSS-Protection
        * Strict-Transport-Security
        * X-Frame-Options
        * Content-Security-Policy ```
</pre>

<pre> ```js 6️⃣ Enable Rate Limiting to Stop Brute Force ``` </pre>
 
 <div style="background:#1e1e1e; color:#fff; padding:15px; border-radius:8px; font-family:monospace;">
  <pre><code>
    const rateLimit = require('hapi-rate-limit');
    await server.register({
    plugin: rateLimit,
    options: {
            userLimit: 100, // per minute
            userCache: {
                expiresIn: 60000
            }
        }
    });
  </code></pre>
</div>

<pre> ```js 7️⃣ Enable CORS Securely ``` </pre>

<div style="background:#1e1e1e; color:#fff; padding:15px; border-radius:8px; font-family:monospace;">
  <pre><code>
    const server = Hapi.server({
    port: 4000,
    host: 'localhost',
    routes: {
            cors: {
            origin: ['*'], // Replace '*' with specific origins in production
                additionalHeaders: ['authorization']
            }
        }
    });
  </code></pre>
</div>

<pre> ```js 8️⃣ Use HTTPS in Production (via Nginx or AWS ELB)``` </pre>

    (```Use a reverse proxy like Nginx to terminate SSL.```)

    (```Always redirect HTTP → HTTPS.```)

