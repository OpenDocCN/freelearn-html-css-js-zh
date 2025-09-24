# 第六章：使用`passport.js`构建认证

认证是任何应用程序的重要组成部分。认证是保护我们构建的应用程序的一种方式。每个应用程序都需要某种认证机制。它帮助我们识别向应用程序服务器发送请求的用户。

在本章中，我们将讨论以下主题：

+   创建登录和注册页面

+   安装和配置`passport.js`

+   了解更多关于`passport.js`策略，即**JSON Web Token**（JWT）策略

+   了解更多关于`passport.js`本地策略

+   在应用程序服务器中创建必要的端点以处理注册和登录请求

我们可以自己构建用户认证。然而，这需要大量的配置和很多麻烦。`passport.js`是一个允许我们高效配置认证的包，只需花费很少的时间。如果您想自己学习和开发，我鼓励您这样做。这将使您更深入地了解一切是如何工作的。然而，对于这本书，我们将使用这个叫做`passport.js`的强大工具，它非常容易集成和学习。

到目前为止，我们已经创建了一个动态的 Web 应用程序，该程序在主页上显示我们通过电影添加表单和 API 添加的所有电影。我们也有一种方法通过前端将这些电影添加到数据库中。现在，由于这将是一个公开的 Web 应用程序，我们不能允许每个人在未登录的情况下自行添加电影。只有登录的用户才能访问并添加电影。此外，为了对电影进行评分，用户应先登录，然后对电影进行评分。

# `passport.js`简介

`passport.js`是 Node.js 提供的用于认证的中间件。`passport.js`的功能是认证发送到服务器的请求。它提供了多种认证策略。`passport.js`提供了诸如本地策略、Facebook 策略、Google 策略、Twitter 策略和 JWT 策略等策略。在本章中，我们将专注于使用 JWT 策略。

# JWT

JWT 是一种使用基于令牌的方法进行请求认证的方式。有两种认证请求的方法：基于 cookie 的认证和基于令牌的认证。基于 cookie 的认证机制将用户的会话 ID 保存在浏览器的 cookie 中，而基于令牌的机制使用一个签名的令牌，其外观如下：

```js
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVhNjhhNDMzMDJkMWNlZDU5YjExNDg3MCIsImlhdCI6MTUxNzI0MjM1M30.5xY59iTIjpt9ukDmxseNAGbOdz6weWL1drJkeQzoO3M
```

然后，这个令牌将在我们向`controllers`发出的每个请求上进行验证。

对于我们的应用程序，我们将使用两种方法的组合。当用户请求登录到应用程序时，我们将为他们创建一个签名的令牌，然后将该令牌添加到浏览器的 cookie 中。下次用户登录时，我们将从 cookie 中读取该令牌，并使用服务器中的`passport-jwt`模块验证该令牌，然后决定是否让该用户登录。

如果你仔细查看前面的令牌，你会看到令牌由三个部分组成，由句点(`.`)分隔；每个部分都有自己的含义：

+   第一部分代表头部

+   第二部分代表有效载荷

+   第三部分代表签名

为了能够使用这个 JWT，我们需要添加一个包。为此，我们可以运行以下命令：

```js
$ npm install jsonwebtoken --save

```

要开始使用这个包，让我们在`server.js`中也定义它：

```js
...
const morgan = require('morgan')
const fs = require('fs')
const jwt = require('jsonwebtoken');
...
```

# 安装 passport.js

就像任何其他的`npm`包一样，我们可以通过运行以下命令来安装`passport.js`：

```js
$ npm install passport --save
```

在成功安装后，你应该在你的`package.json`文件中列出以下这些包：

```js
...
"nodemon": "¹.14.10",
"passport": "⁰.4.0",
"sass-loader": "⁶.0.6",
...
```

你也可以通过首先将包添加到你的`package.json`文件中，然后运行以下命令来完成此操作：

```js
$ npm install
```

# 配置 passport

就像任何其他的`node`包一样，我们需要为`passport.js`配置包。在我们的`server.js`文件中，添加以下几行代码：

```js
...
const mongoose = require('mongoose');
const cors = require('cors');
const morgan = require('morgan');
const fs = require('fs');
const jwt = require('jsonwebtoken');
const passport = require('passport');

const app = express();
const router = express.Router();
app.use(morgan('combined'));
app.use(bodyParser.json());
app.use(cors());
app.use(passport.initialize());
...
```

上述代码只是在我们应用程序中初始化了`passport.js`。我们仍然需要配置一些东西才能开始使用 JWT 认证机制。

# passport.js 策略

如前所述，`passport.js`提供了许多策略以实现轻松集成。我们将要使用的一个策略是 JWT 策略。我们已经添加了`passport.js`并初始化了它。现在，让我们也添加这个策略。

# 安装 passport-jwt 策略

仅安装 passport 模块并不足以满足我们的需求。`passport.js`提供了其策略在单独的`npm`包中。对于`jwt`认证，我们必须安装`passport-jwt`模块，如下所示：

```js
$ npm install passport-jwt --save
```

在成功安装后，你应该在应用程序的`package.json`文件中列出以下这些包：

```js
...
"nodemon": "¹.14.10",
"passport": "⁰.4.0", "passport-jwt": "³.0.1",
"sass-loader": "⁶.0.6",
...

```

# 配置 passport-jwt 策略

现在我们已经拥有了所有需要的东西，让我们跳转到 JWT 策略的配置设置。在`server.js`中添加以下几行代码：

```js
...
const morgan = require('morgan');
const fs = require('fs');
const jwt = require('jsonwebtoken');
const passport = require('passport');
const passportJWT = require('passport-jwt');
const ExtractJwt = passportJWT.ExtractJwt;
const JwtStrategy = passportJWT.Strategy;
const jwtOptions = {}
jwtOptions.jwtFromRequest = ExtractJwt.fromAuthHeaderWithScheme('jwt');
jwtOptions.secretOrKey = 'movieratingapplicationsecretkey';

const app = express();
const router = express.Router();
...
```

上述代码足以让我们开始。我们需要从`passport.js`中获取`JwtStrategy`，并且`ExtractJwT`将用于从`jwt`令牌中提取有效载荷数据。

我们还定义了一个变量来设置 JWT `auth`设置，其中配置了一个密钥。这个密钥将用于签名任何请求的有效载荷。

你也可以创建一个单独的文件来存储你的重要密钥。

# 使用 JWT 策略

现在我们已经准备好使用`passport.js`提供的服务了。让我们快速回顾一下到目前为止我们已经做了什么：

1.  安装了 passport、`passport-jwt`和`jsonwebtoken`

1.  为这三个包配置了所有设置

下一步如下：

1.  创建我们的用户模型

1.  为用户实体创建 API 端点，即登录和注册

1.  构建我们的认证视图，即登录页面和注册页面

1.  使用 JWT 策略最终验证请求

# 设置用户注册

让我们从添加用户注册功能开始。

# 创建用户模型

我们还没有一个集合来管理用户。在我们的 `User` 模型中将有三个参数：`name`、`email` 和 `password`。让我们继续在 `models` 目录中创建我们的 `User` 模型，名为 `User.js`：

```js
const mongoose = require('mongoose');

const Schema = mongoose.Schema;
const UserSchema = new Schema({
 name: String,
 email: String,
 password: String,
});

const User = mongoose.model('User', UserSchema);
module.exports = User;
```

如您所见，以下是为用户定义的三个属性：`name`、`email` 和 `password`。

# 安装 bcryptjs

现在，我们不能再以明文形式保存这些用户的密码，因此我们需要一个机制来加密它们。幸运的是，我们已经有了一个专门用于加密密码的包，即 `bcryptjs`。让我们首先将这个包添加到我们的应用程序中：

```js
$ npm install bcryptjs --save
```

当包安装完成后，让我们在 `User.js` 模型中添加初始化块：

```js
const mongoose = require('mongoose');
const bcryptjs = require('bcryptjs');

const Schema = mongoose.Schema;
const UserSchema = new Schema({
  name: String,
  email: String,
  password: String,
});

const User = mongoose.model('User', UserSchema);
module.exports = User;
```

现在，当我们保存一个用户时，我们应该创建我们自己的方法来将用户添加到数据库中，因为我们想加密他们的密码。所以，让我们将以下代码添加到 `models/User.js` 中：

```js
...
const User = mongoose.model('User', UserSchema);
module.exports = User;

module.exports.createUser = (newUser, callback) => {
 bcryptjs.genSalt(10, (err, salt) => {
 bcryptjs.hash(newUser.password, salt, (error, hash) => {
 // store the hashed password
 const newUserResource = newUser;
 newUserResource.password = hash;
 newUserResource.save(callback);
 });
 });
};
...
```

在前面的代码中，我们使用了 `bcrypt` 库，该库使用 `genSalt` 机制将密码转换为加密字符串。在 `User` 模型中的 preceding 方法 `createUser` 接收 `user` 对象，将用户提供的密码转换为加密密码，并将其保存到数据库中。

# 为注册用户添加 API 端点

现在我们已经有了我们的模型，让我们继续创建一个用于创建用户的端点。为此，让我们首先在 `controllers` 文件夹中创建一个名为 `users.js` 的控制器，以管理所有与用户相关的请求。由于我们在 `server.js` 中添加了一个代码块来初始化 `controllers` 目录中的所有文件，所以我们在这里不需要引入这些文件。

在 `users.js` 中，将文件内容替换为以下代码：

```js
const User = require('../models/User.js');

module.exports.controller = (app) => {
 // register a user
 app.post('/users/register', (req, res) => {
 const name = req.body.name;
 const email = req.body.email;
 const password = req.body.password;
 const newUser = new User({
 name,
 email,
 password,
 });
 User.createUser(newUser, (error, user) => {
 if (error) { console.log(error); }
 res.send({ user });
 });
 });
};
```

在前面的代码中，我们添加了一个端点，该端点向 `http://localhost:8081/users/register` URL 发送 POST 请求，获取用户的 `name`、`email` 和 `password`，并将它们保存到我们的数据库中。在响应中，它返回刚刚创建的用户。这很简单。

现在，让我们在 Postman 中测试这个端点。您应该能够在响应中看到返回的用户：

![](img/64757a12-83f6-4ab9-a3d9-a61e0f19b6fd.png)

# 创建注册视图页面

让我们为用户添加一个注册视图页面。为此，我们需要创建一个表单，该表单接受 `name`、`email` 和 `password` 参数。在 `src/components` 中创建一个名为 `Register.vue` 的文件：

```js
<template>
 <v-form v-model="valid" ref="form" lazy-validation>
 <v-text-field
 label="Name"
 v-model="name"
 required
 ></v-text-field>
 <v-text-field
 label="Email"
 v-model="email"
 :rules="emailRules"
 required
 ></v-text-field>
 <v-text-field
 label="Password"
 v-model="password"
 required
 ></v-text-field>
 <v-text-field
 name="input-7-1"
 label="Confirm Password"
 v-model="confirm_password"
 ></v-text-field>
 <v-btn
 @click="submit"
 :disabled="!valid"
 >
 submit
 </v-btn>
 <v-btn @click="clear">clear</v-btn>
 </v-form>
</template>
```

`vue` 文件是一个简单的模板文件，其中包含表单组件。下一步是为该文件添加路由。

在 `src/router/index.js` 中，添加以下代码行：

```js
import Vue from 'vue';
import Router from 'vue-router';
import Home from '@/components/Home';
import Contact from '@/components/Contact';
import AddMovie from '@/components/AddMovie';
import Movie from '@/components/Movie';
import Register from '@/components/Register';

Vue.use(Router);

export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'Home',
      component: Home,
    },
    {
      path: '/contact',
      name: 'Contact',
      component: Contact,
    },
    {
      path: '/movies/add',
      name: 'AddMovie',
      component: AddMovie,
    },{ path: '/movies/:id',name: 'Movie',component: Movie,},
 {
 path: '/users/register',
 name: 'Register',
 component: Register,
 },
  ],
});
```

就这样！现在，让我们导航到 `http://localhost.com:8080/users/register`：

![](img/45360b7b-0695-4de3-a24c-25b3bcdf6448.png)

# 在注册表单中添加提交和清除方法

下一步是为 `submit` 和 `clear` 方法添加功能。让我们在 `Register.vue` 中添加一些方法：

```js
...
    <v-btn @click="clear">clear</v-btn>
  </v-form>
</template>
<script>
export default {
 data: () => ({
 valid: true,
 name: '',
 email: '',
 password: '',
 confirm_password: '',
 emailRules: [
 v => !!v || 'E-mail is required',
 v => /\S+@\S+\.\S+/.test(v) || 'E-mail must be valid',
 ],
 }),
 methods: {
 async submit() {
 if (this.$refs.form.validate()) {
 // add process here
 }
 },
 clear() {
 this.$refs.form.reset();
 },
 },
};
</script>
```

我们还在此处添加了一些注册表单的验证。它根据给定的正则表达式验证用户提供的电子邮件。

我们添加了两个方法，`submit`和`clear`。`clear`方法重置表单值；相当直接，对吧？现在，当我们点击`submit`按钮时，首先运行验证。如果所有验证都通过，那么只有`submit`方法内的逻辑会被处理。在这里，我们需要向服务器发送带有用户参数的请求，这时`axios`就派上用场了。

# 介绍 axios

Axios 是一种将请求数据发送到服务器的机制。你可以将其视为 JavaScript 中的 AJAX 请求。使用`axios`，我们可以有效地处理来自服务器的成功和错误响应。

要安装`axios`，请运行以下命令：

```js
$ npm install axios --save
```

# 使用 axios

现在，让我们修改`Register.vue`文件以实现`axios`——按照以下方式替换`script`标签内的内容：

```js
...
</v-form>
</template>
<script>
import axios from 'axios';

export default {
  data: () => ({
    valid: true,
    name: '',
    email: '',
    password: '',
    confirm_password: '',
    emailRules: [
      v => !!v || 'E-mail is required',
      v => /\S+@\S+\.\S+/.test(v) || 'E-mail must be valid',
    ],
  }),
  methods: {
    async submit() {
 if (this.$refs.form.validate()) {
 return axios({
 method: 'post',
 data: {
 name: this.name,
 email: this.email,
 password: this.password,
 },
 url: 'http://localhost:8081/users/register',
 headers: {
 'Content-Type': 'application/json',
 },
 })
 .then(() => {
 this.$swal(
 'Great!',
 'You have been successfully registered!',
 'success',
 );
 this.$router.push({ name: 'Login' });
 })
 .catch((error) => {
 const message = error.response.data.message;
 this.$swal('Oh oo!', `${message}`, 'error');
 });
 }
 return true;
 },
 clear() {
 this.$refs.form.reset();
 },
  },
};
</script>
```

如果你熟悉`ajax`，你应该能够快速理解代码。如果不熟悉，别担心，实际上它相当简单。`axios`方法接受重要的参数，例如请求方法（在前述情况中为`post`）、数据参数或有效载荷，以及要击中的 URL 端点。它接受这些参数并将它们路由到`then()`方法或`catch()`方法，具体取决于服务器的响应。

如果请求成功，它将进入`then()`方法；如果不成功，它将进入`catch()`方法。现在，请求的成功和失败也可以根据我们的需求进行自定义。对于前述场景，如果用户没有保存到数据库中，我们将简单地传递一个错误响应。我们也可以对验证进行操作。

因此，让我们也修改`controller`方法中的`users.js`以适应这些更改：

```js
const User = require('../models/User.js');

module.exports.controller = (app) => {
  // register a user
  app.post('/users/register', (req, res) => {
    const name = req.body.name;
    const email = req.body.email;
    const password = req.body.password;
    const newUser = new User({
      name,
      email,
      password,
    });
    User.createUser(newUser, (error, user) => {
      if (error) {
 res.status(422).json({
 message: 'Something went wrong. Please try again after some time!',
 });
 }
      res.send({ user });
    });
  });
};
```

如前述代码所示，如果请求失败，我们将发送一条消息说“出了点问题”。我们还可以根据服务器的响应显示不同类型的消息。

# 设置用户登录

现在我们已经成功实现了用户的登录过程，接下来让我们开始构建将用户登录到我们应用的功能。

# 修改用户模型

要将用户登录到应用中，我们将使用以下两个参数：用户的电子邮件和他们的密码。我们需要查询数据库以找到给定电子邮件的记录；因此，让我们添加一个方法来根据用户名提取用户：

```js
...
const User = mongoose.model('User', UserSchema);
module.exports = User;

module.exports.createUser = (newUser, callback) => {
  bcryptjs.genSalt(10, (err, salt) => {
    bcryptjs.hash(newUser.password, salt, (error, hash) => {
      // store the hashed password
      const newUserResource = newUser;
      newUserResource.password = hash;
      newUserResource.save(callback);
    });
  });
};

module.exports.getUserByEmail = (email, callback) => {
 const query = { email };
 User.findOne(query, callback);
};
```

前述方法将返回具有给定电子邮件的用户。

正如我提到的，我们还需要检查的另一件事是密码。让我们添加一个方法来比较用户在登录时提供的密码和保存在我们数据库中的密码：

```js
...
module.exports.getUserByEmail = (email, callback) => {
  const query = { email };
  User.findOne(query, callback);
};

module.exports.comparePassword = (candidatePassword, hash, callback) => {
 bcryptjs.compare(candidatePassword, hash, (err, isMatch) => {
 if (err) throw err;
 callback(null, isMatch);
 });
};
```

前述方法同时接受用户提供的密码和保存的密码，并根据密码是否匹配返回`true`或`false`。

现在我们已经准备好进入控制器部分。

# 添加用于登录用户的 API 端点

我们已经添加了用户能够登录所需的方法。现在，本章最重要的部分就在这里。我们需要设置 JWT `auth`机制，以使用户能够登录。

在`users.js`中添加以下代码行：

```js
const User = require('../models/User.js');

const passportJWT = require('passport-jwt');
const jwt = require('jsonwebtoken');

const ExtractJwt = passportJWT.ExtractJwt;
const jwtOptions = {};
jwtOptions.jwtFromRequest = ExtractJwt.fromAuthHeaderWithScheme('jwt');
jwtOptions.secretOrKey = 'thisisthesecretkey';

module.exports.controller = (app) => {
  // register a user
  app.post('/users/register', (req, res) => {
    const name = req.body.name;
    const email = req.body.email;
    const password = req.body.password;
    const newUser = new User({
      name,
      email,
      password,
    });
    User.createUser(newUser, (error, user) => {
      if (error) {
        res.status(422).json({
          message: 'Something went wrong. Please try again after some time!',
        });
      }
      res.send({ user });
    });
  });

  // login a user
 app.post('/users/login', (req, res) => {
 if (req.body.email && req.body.password) {
 const email = req.body.email;
 const password = req.body.password;
 User.getUserByEmail(email, (err, user) => {
 if (!user) {
 res.status(404).json({ message: 'The user does not exist!' });
 } else {
 User.comparePassword(password, user.password, (error, isMatch) => {
 if (error) throw error;
 if (isMatch) {
 const payload = { id: user.id };
 const token = jwt.sign(payload, jwtOptions.secretOrKey);
 res.json({ message: 'ok', token });
 } else {
 res.status(401).json({ message: 'The password is incorrect!' });
 }
 });
 }
 });
 }
 });
};
```

由于 JWT 策略是`passport.js`的一部分，我们还需要初始化它。我们还需要为 JWT 选项添加一些配置，以从负载中提取数据，并在请求发送到服务器时对其进行解密和重新加密。

密钥是你可以配置的东西。它基本上代表了你的应用令牌。确保它不容易被猜到。

此外，我们还添加了一个端点，它向`localhost:8081/users/login`发送 POST 请求，并接收用户的电子邮件和密码。以下是这个方法所做的一些事情：

+   检查给定电子邮件的用户是否存在。如果不存在，它将发送一个 404 状态码，指出用户在我们的应用中不存在。

+   将提供的密码与我们的应用中用户的密码进行比较。如果没有匹配，它将发送一个错误响应，指出密码不匹配。

+   如果一切顺利，它将使用 JWT 签名对用户的负载进行签名，生成一个令牌，并响应该令牌。

现在，让我们在 Postman 中测试这个端点。你应该能够在响应中看到返回的令牌，如下所示：

![图片](img/9d43f252-4443-4cdb-b915-48e3fd30c63f.png)

在前面的屏幕截图中，请注意 JWT 接受负载，对其进行签名，并生成一个随机令牌。

# 创建注册视图页面

现在让我们为用户添加一个登录页面。为此，就像我们在注册页面所做的那样，我们需要创建一个包含电子邮件和密码参数的表单。在`src/components`目录内创建一个名为`Login.vue`的文件，如下所示：

```js
<template>
 <v-form v-model="valid" ref="form" lazy-validation>
 <v-text-field
 label="Email"
 v-model="email"
 :rules="emailRules"
 required
 ></v-text-field>
 <v-text-field
 label="Password"
 v-model="password"
 required
 ></v-text-field>
 <v-btn
 @click="submit"
 :disabled="!valid"
 >
 submit
 </v-btn>
 <v-btn @click="clear">clear</v-btn>
 </v-form>
</template>
```

`vue`文件是一个简单的模板文件，其中包含表单组件。接下来要做的事情是为该文件添加一个路由。

在`src/router/index.js`中添加以下代码：

```js
import Vue from 'vue';
import Router from 'vue-router';
import Home from '@/components/Home';
import Contact from '@/components/Contact';
import AddMovie from '@/components/AddMovie';
import Movie from '@/components/Movie';
import Register from '@/components/Register';
import Login from '@/components/Login';

Vue.use(Router);

export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'Home',
      component: Home,
    },
    {
      path: '/contact',
      name: 'Contact',
      component: Contact,
    },
    {
      path: '/movies/add',
      name: 'AddMovie',
      component: AddMovie,
    },
    {
      path: '/movies/:id',
      name: 'Movie',
      component: Movie,
    },
    {
      path: '/users/register',
      name: 'Register',
      component: Register,
    },
    {
 path: '/users/login',
 name: 'Login',
 component: Login,
 },
  ],
});
```

就这样。现在，让我们导航到`http://localhost.com:8080/users/login`：

![图片](img/36840d36-9b09-4219-9797-57ad3e10df90.png)

# 在登录表单中添加提交和清除方法

下一步是在`submit`和`clear`方法中添加功能。让我们在`Login.vue`中添加一些方法。`clear`方法与注册页面上的相同。对于`submit`方法，我们将在这里使用`axios`方法。我们已经在控制器中分类了我们的成功和错误消息。现在我们只需要确保它们在 UI 中显示：

```js
...
</v-form>
</template>
<script>
import axios from 'axios';

export default {
 data: () => ({
 valid: true,
 email: '',
 password: '',
 emailRules: [
 v => !!v || 'E-mail is required',
 v => /\S+@\S+\.\S+/.test(v) || 'E-mail must be valid',
 ],
 }),
 methods: {
 async submit() {
 return axios({
 method: 'post',
 data: {
 email: this.email,
 password: this.password,
 },
 url: 'http://localhost:8081/users/login',
 headers: {
 'Content-Type': 'application/json',
 },
 })
 .then((response) => {
 window.localStorage.setItem('auth', response.data.token);
 this.$swal('Great!', 'You are ready to start!', 'success');
 this.$router.push({ name: 'Home' });
 })
 .catch((error) => {
 const message = error.response.data.message;
 this.$swal('Oh oo!', `${message}`, 'error');
 this.$router.push({ name: 'Login' });
 });
 },
 clear() {
 this.$refs.form.reset();
 },
 },
};
</script>
```

验证与注册页面上的相同。我们添加了两个方法，`submit`和`clear`。`clear`方法重置表单值，而`submit`方法只是简单地调用 API 端点，从表单中获取参数，并响应正确的消息，然后该消息在 UI 中显示。成功完成后，用户将被重定向到主页。

这里重要的是，由于我们在客户端进行交互，我们需要将之前生成的 JWT 令牌保存到某个地方。访问令牌的最佳方式是将它保存到浏览器的会话中。因此，我们设置了一个名为`auth`的键，它将 JWT 令牌保存在本地存储中。每当发出其他请求时，请求将首先检查它是否是一个有效的令牌，然后相应地执行操作。

以下是我们迄今为止所做的工作：

+   在用户模型中添加了`getUserByEmail()`和`comparePassword()`

+   创建了登录视图页面

+   添加了提交和清除表单的方法

+   生成 JWT 签名令牌并将其保存到会话中以便以后重用

+   显示成功和错误消息

# 在 Home.vue 中验证我们的用户

我们需要做的最后一件事是检查当前登录用户是否有权查看电影列表页面。虽然让主页（电影列表页面）对所有用户开放是有意义的，但为了学习目的，当用户访问主页时，让我们添加 JWT 授权。让我们让主页不对不在我们应用中的外部用户开放。

在`movies.js`中添加以下代码段：

```js
const MovieSchema = require('../models/Movie.js');
const Rating = require('../models/Rating.js');
const passport = require('passport');

module.exports.controller = (app) => {
  // fetch all movies
  app.get('/movies', passport.authenticate('jwt', { session: false }), (req, res) => {
    MovieSchema.find({}, 'name description release_year genre', (error, movies) => {
      if (error) { console.log(error); }
      res.send({
        movies,
      });
    });
  });
...
```

好的，就是这样！我们需要初始化 passport 并添加`passport.authenticate('jwt', { session: false })`。我们必须传递 JWT 令牌，passport JWT 策略会自动验证当前用户。

现在，让我们在请求电影列表页面时也发送 JWT 令牌。在`Home.vue`中添加以下代码：

```js
...
<script>
import axios from 'axios';

export default {
  name: 'Movies',
  data() {
    return {
      movies: [],
    };
  },
  mounted() {
    this.fetchMovies();
  },
  methods: {
    async fetchMovies() {
 const token = window.localStorage.getItem('auth');
 return axios({
 method: 'get',
 url: 'http://localhost:8081/movies',
 headers: {
 Authorization: `JWT ${token}`,
 'Content-Type': 'application/json',
 },
 })
 .then((response) => {
 this.movies = response.data.movies;
 this.current_user = response.data.current_user;
 })
 .catch(() => {
 });
 },
  },
};
</script>
```

在进行`axios`调用时，我们将在头部传递一个额外的参数。我们需要从本地存储中读取令牌并将其通过头部传递给电影 API。

使用这种方式，任何未登录到应用的用户将无法查看电影列表页面。

# 为 Vue 组件提供静态文件服务

在跳入本地策略之前，让我们了解一下如何使我们的 Vue.js 组件以静态方式提供服务。由于我们使用的是独立的前端和后端，维护这两个版本可能会是一项艰巨的任务，尤其是在部署应用时，配置一切可能需要更长的时间。因此，为了更好地管理我们的应用，我们将构建 Vue.js 应用，这将是一个生产构建，并且仅使用 Node.js 服务器来提供文件。为此，我们将使用一个名为`serve-static`的独立包。所以，让我们继续安装这个包：

```js
$ npm install serve-static --save 
```

现在，让我们将以下内容添加到我们的`server.js`文件中：

```js
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const cors = require('cors');
const morgan = require('morgan');
const fs = require('fs');
const session = require('express-session');
const config = require('./config/Config');
const passport = require('passport');
const app = express();
const router = express.Router();
const serveStatic = require('serve-static');

app.use(morgan('combined'));
app.use(bodyParser.json());
app.use(cors());

...

// Include controllers
fs.readdirSync("controllers").forEach(function (file) {
  if(file.substr(-3) == ".js") {
    const route = require("./controllers/" + file)
    route.controller(app)
  }
})
app.use(serveStatic(__dirname + "/dist"));
...
```

使用这个命令，现在让我们构建我们的应用：

```js
$ npm run build 
```

之前的命令将在应用内的`dist`文件夹中创建必要的静态文件，这些文件将由运行在 8081 端口的 Node.js 服务器提供。构建完成后，我们现在不需要运行以下命令：

```js
$ npm run dev 
```

此外，现在由于我们只运行 node 服务器，应用应该可以通过`http://localhost:8081`的 URL 访问。

上述命令启动了我们的前端服务器。我们只需要使用以下命令运行 Node.js 服务器：

```js
$ nodemon server.js
```

由于我们现在只有一个端口，8081，我们不需要在像之前那样在每个后端 API 中添加`/api`前缀，我们可以去掉这些前缀。因此，让我们更新`controllers`和`vue`文件：

如下替换`controllers/movies.js`中的内容：

```js
var Movie = require("../models/Movie");

module.exports.controller = (app) => {
  // fetch all movies
 app.get("/movies", function(req, res) {
    Movie.find({}, 'name description release_year genre', function 
    (error, movies) {
      if (error) { console.log(error); }
       res.send({
        movies: movies
      })
    })
  })

  // add a new movie
 app.post('/movies', (req, res) => {
    const movie = new Movie({
      name: req.body.name,
      description: req.body.description,
      release_year: req.body.release_year,
      genre: req.body.genre
    })

    movie.save(function (error, movie) {
      if (error) { console.log(error); }
      res.send(movie)
    })
  })
}
```

如下替换`controllers/users.js`中的内容：

```js
const User = require("../models/User");
const config = require('./../config/Config');
const passport = require('passport');

module.exports.controller = (app) => {
  // local strategy
  const LocalStrategy = require('passport-local').Strategy;
  passport.use(new LocalStrategy({
      usernameField: 'email',
      passwordField: 'password'
    },
    function(email, password, done) {
      User.getUserByEmail(email, function(err, user){
        if (err) { return done(err); }
        if (!user) { return done(null, false); }
        User.comparePassword(password, user.password, function(err, 
        isMatch){
          if(isMatch) {
            return done(null, user);
          } else {
            return done(null, false);
          }
        })
      });
    }
  ));

 app.post('/users/login',
    passport.authenticate('local', { failureRedirect: '/users/login' }),
    function(req, res) {
      res.redirect('/');
    });

  passport.serializeUser(function(user, done) {
    done(null, user.id);
  });

  passport.deserializeUser(function(id, done) {
    User.findById(id, function(err, user){
      done(err, user)
    })
  });

  // register a user
 app.post('/users/register', (req, res) => {
    const email = req.body.email;
    const fullname = req.body.fullname;
    const password = req.body.password;
    const role = req.body.role || 'user';
    const newUser = new User({
      email: email,
      fullname: fullname,
      role: role,
      password: password
    })
    User.createUser(newUser, function(error, user) {
      if (error) {
        res.status(422).json({
          message: "Something went wrong. Please try again after some 
          time!"
        });
      }
      res.send({ user: user })
    })
  })
}
```

将`AddMovie.vue`的`script`标签内容替换为以下代码：

```js
<script>
import axios from 'axios';

export default {
  data: () => ({
    valid: true,
    name: '',
    description: '',
    genre: '',
    release_year: '',
    nameRules: [
      v => !!v || 'Movie name is required',
    ],
    genreRules: [
      v => !!v || 'Movie genre year is required',
      v => (v && v.length <= 80) || 'Genre must be less than equal to 
      80 characters.',
    ],
    releaseRules: [
      v => !!v || 'Movie release year is required',
    ],
    select: null,
    years: [
      '2018',
      '2017',
      '2016',
      '2015',
    ],
  }),
  methods: {
    submit() {
      if (this.$refs.form.validate()) {
        return axios({
          method: 'post',
          data: {
            name: this.name,
            description: this.description,
            release_year: this.release_year,
            genre: this.genre,
          },
 url: '/movies',
          headers: {
            'Content-Type': 'application/json',
          },
        })
          .then(() => {
            this.$swal(
              'Great!',
              'Movie added successfully!',
              'success',
            );
            this.$router.push({ name: 'Home' });
            this.$refs.form.reset();
          })
          .catch(() => {
            this.$swal(
              'Oh oo!',
              'Could not add the movie!',
              'error',
            );
          });
      }
      return true;
    },
    clear() {
      this.$refs.form.reset();
    },
  },
};
</script>
```

将`Home.vue`的`script`标签内容替换为以下代码：

```js
<script>
import axios from 'axios';

export default {
  name: 'Movies',
  data() {
    return {
      movies: [],
    };
  },
  mounted() {
    this.fetchMovies();
  },
  methods: {
    async fetchMovies() {
      return axios({
        method: 'get',
 url: '/movies',
      })
        .then((response) => {
          this.movies = response.data.movies;
        })
        .catch(() => {
        });
    },
  },
};
</script>
```

将`Login.vue`的`script`标签内容替换为以下代码：

```js
<script>
  import axios from 'axios';
  import bus from "./../bus.js";

  export default {
    data: () => ({
      valid: true,
      email: '',
      password: '',
      emailRules: [
        (v) => !!v || 'E-mail is required',
        (v) => /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(v) 
        || 'E-mail must be valid'
      ],
      passwordRules: [
        (v) => !!v || 'Password is required',
      ]
    }),
    methods: {
      async submit () {
        if (this.$refs.form.validate()) {
          return axios({
            method: 'post',
            data: {
              email: this.email,
              password: this.password
            },
 url: '/users/login',
            headers: {
              'Content-Type': 'application/json'
            }
          })
          .then((response) => {
            localStorage.setItem('jwtToken', response.data.token)
            this.$swal("Good job!", "You are ready to start!", 
            "success");
            bus.$emit("refreshUser");
            this.$router.push({ name: 'Home' });
          })
          .catch((error) => {
            const message = error.response.data.message;
            this.$swal("Oh oo!", `${message}`, "error")
          });
        }
      },
      clear () {
        this.$refs.form.reset()
      }
    }
  }
</script>
```

将`Register.vue`的`script`标签内容替换为以下代码：

```js
<script>
  import axios from 'axios';
  export default {
    data: () => ({
      e1: false,
      valid: true,
      fullname: '',
      email: '',
      password: '',
      confirm_password: '',
      fullnameRules: [
        (v) => !!v || 'Fullname is required'
      ],
      emailRules: [
        (v) => !!v || 'E-mail is required',
        (v) => /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(v) 
        || 'E-mail must be valid'
      ],
      passwordRules: [
        (v) => !!v || 'Password is required'
      ]
    }),
    methods: {
      async submit () {
        if (this.$refs.form.validate()) {
          return axios({
            method: 'post',
            data: {
              fullname: this.fullname,
              email: this.email,
              password: this.password
            },
 url: '/users/register',
            headers: {
              'Content-Type': 'application/json'
            }
          })
          .then((response) => {
            this.$swal(
              'Great!',
              `You have been successfully registered!`,
              'success'
            )
            this.$router.push({ name: 'Home' })
          })
          .catch((error) => {
            const message = error.response.data.message;
            this.$swal("Oh oo!", `${message}`, "error")
          });
        }
      },
      clear () {
        this.$refs.form.reset()
      }
    }
  }
</script>
```

最后，我们不再需要使用代理，因此我们可以从`webpack.dev.conf.js`中移除我们之前设置的代理。

如下替换`devServer`中的内容：

```js
devServer: {
    clientLogLevel: 'warning',
    historyApiFallback: {
      rewrites: [
        { from: /.*/, to: path.posix.join(config.dev.assetsPublicPath, 
        'index.html') },
      ],
    },
    hot: true,
    contentBase: false, // since we use CopyWebpackPlugin.
    compress: true,
    host: HOST || config.dev.host,
    port: PORT || config.dev.port,
    open: config.dev.autoOpenBrowser,
    overlay: config.dev.errorOverlay
      ? { warnings: false, errors: true }
      : false,
    publicPath: config.dev.assetsPublicPath,
    quiet: true, // necessary for FriendlyErrorsPlugin
    watchOptions: {
      poll: config.dev.poll,
    }
  },
```

更新后，让我们使用以下命令再次构建我们的应用程序：

```js
$ npm run build
```

我们的应用程序应该按预期工作。

由于我们的应用程序是一个**单页应用程序（SPA**），当我们浏览嵌套路由并重新加载页面时，我们会得到一个错误。例如，如果我们通过点击主页中的链接浏览`http://localhost:8081/contact`页面，它会正常工作。然而，如果我们直接尝试导航到`http://localhost:8081/contact`页面，我们会得到一个错误，因为这是一个 SPA，这意味着浏览器只渲染静态的`index.html`文件。当我们尝试访问`/contact`页面时，它会寻找名为`contact`的页面，而这个页面不存在。

为了实现这一点，我们需要添加一个中间件，它作为一个回退，在我们尝试直接重新加载页面或尝试访问具有动态 ID 的页面时，渲染相同的`index.html`文件。

`npm`提供了一个中间件来满足我们的需求。让我们继续安装以下包：

```js
$ npm install connect-history-api-fallback --save
```

安装完成后，让我们修改我们的`server.js`文件以使用中间件：

```js
...
const passport = require('passport');
const serveStatic = require('serve-static');
const history = require('connect-history-api-fallback');
const app = express();
const router = express.Router();

...

// Include controllers
fs.readdirSync("controllers").forEach(function (file) {
  if(file.substr(-3) == ".js") {
    const route = require("./controllers/" + file)
    route.controller(app)
  }
})
app.use(history());
app.use(serveStatic(__dirname + "/dist"));
...
```

在这些设置到位后，我们现在应该能够直接访问所有路由。我们也可以现在重新加载页面。

由于我们仅在 Node.js 服务器上构建 Vue.js 组件并运行应用程序，因此每当我们对 Vue.js 组件进行更改时，我们都需要使用`npm run build`命令再次构建应用程序。

# Passport 的本地策略

Passport 的本地策略很容易集成。一如既往，让我们从以下命令开始安装这个策略。

# 安装 Passport 的本地策略

我们可以通过运行以下命令安装 Passport 的本地策略：

```js
$ npm install passport-local --save
```

以下代码应将包添加到您的`package.json`文件中：

```js
...
"node-sass": "⁴.7.2",
"nodemon": "¹.14.10",
"passport": "⁰.4.0",
"passport-local": "¹.0.0",
...
```

# 配置 Passport 的本地策略

配置 Passport 的本地策略有几个步骤。我们将详细讨论每个步骤：

1.  添加必要的本地认证路由。

1.  添加一个中间件方法来检查认证是否成功。

让我们深入了解前面步骤的细节。

# 添加必要的本地认证路由

让我们继续添加必要的路由，当我们点击登录按钮时。用以下代码替换`controllers/users.js`的内容：

```js
const User = require('../models/User.js');
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;

module.exports.controller = (app) => {
// local strategy
 passport.use(new LocalStrategy({
 usernameField: 'email',
 passwordField: 'password',
 }, (email, password, done) => {
 User.getUserByEmail(email, (err, user) => {
 if (err) { return done(err); }
 if (!user) { return done(null, false); }
 User.comparePassword(password, user.password, (error, isMatch) => {
 if (isMatch) {
 return done(null, user);
 }
 return done(null, false);
 });
 return true;
 });
 }));

// user login
 app.post('/users/login',
 passport.authenticate('local', { failureRedirect: '/users/login' }),
 (req, res) => {
 res.redirect('/');
 });

 passport.serializeUser((user, done) => {
 done(null, user.id);
 });

 passport.deserializeUser((id, done) => {
 User.findById(id, (err, user) => {
 done(err, user);
 });
 });

  // register a user
  app.post('/users/register', (req, res) => {
    const name = req.body.name;
    const email = req.body.email;
    const password = req.body.password;
    const newUser = new User({
      name,
      email,
      password,
    });
    User.createUser(newUser, (error, user) => {
      if (error) {
        res.status(422).json({
          message: 'Something went wrong. Please try again after some time!',
        });
      }
      res.send({ user });
    });
  });
};
```

在这里，我们添加了一个用户登录的路由`/users/login`，然后使用`passport.js`本地认证机制将用户登录到应用中。

此外，我们配置了`passport.js`，在用户登录时使用 LocalStrategy，它需要用户的`username`和`password`。

# 安装 express-session

下一步，我们需要设置一个`session`，这样当用户成功登录时，`user`数据可以存储在`session`中，并且在我们发出其他请求时可以轻松检索。为此，我们需要添加一个名为`express-session`的包。让我们继续使用以下命令安装这个包：

```js
$ npm install express-session --save
```

# 配置 express-session

现在，我们已经有了这个包，让我们配置这个包以满足我们在`session`中保存用户的需要。在它里面添加以下几行代码。

如果`username`和`password`匹配，用户对象将被保存在服务器的会话中，并且可以通过每个请求中的`req.user`访问。

此外，由于我们现在不需要 passport JWT 策略，让我们也更新我们的 vue 文件。

使用以下代码更新`server.js`中的内容：

```js
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const cors = require('cors');
const morgan = require('morgan');
const fs = require('fs');
const session = require('express-session');
const config = require('./config/Config');
const passport = require('passport');
const serveStatic = require('serve-static');
const history = require('connect-history-api-fallback');

const app = express();
const router = express.Router();
app.use(morgan('combined'));
app.use(bodyParser.json());
app.use(cors());

app.use(session({
 secret: config.SECRET,
 resave: true,
 saveUninitialized: true,
 cookie: { httpOnly: false }
}))
app.use(passport.initialize());
app.use(passport.session());

//connect to mongodb
mongoose.connect(config.DB, function() {
  console.log('Connection has been made');
})
.catch(err => {
  console.error('App starting error:', err.stack);
  process.exit(1);
});

// Include controllers
fs.readdirSync("controllers").forEach(function (file) {
  if(file.substr(-3) == '.js') {
    const route = require('./controllers/' + file);
    route.controller(app);
  }
})
app.use(history());
app.use(serveStatic(__dirname + "/dist"));

router.get('/api/current_user', isLoggedIn, function(req, res) {
 if(req.user) {
 res.send({ current_user: req.user })
 } else {
 res.status(403).send({ success: false, msg: 'Unauthorized.' });
 }
})

function isLoggedIn(req, res, next) {
 if (req.isAuthenticated())
 return next();

 res.redirect('/');
 console.log('error! auth failed')
}

router.get('/api/logout', function(req, res){
 req.logout();
 res.send();
});

router.get('/', function(req, res) {
  res.json({ message: 'API Initialized!'});
});

const port = process.env.API_PORT || 8081;
app.use('/', router);
var server = app.listen(port, function() {
  console.log(`api running on port ${port}`);
});

module.exports = server
```

在这里，我们添加了以下代码块的 express-session 配置：

```js
app.use(session({
 secret: config.SECRET,
 resave: true,
 saveUninitialized: true,
 cookie: { httpOnly: false }
}))
app.use(passport.initialize());
app.use(passport.session());
```

上述代码块使用一个必要的密钥令牌来保存用户详情。我们将在一个单独的文件中定义这个令牌，以便所有配置令牌都位于一个地方。

因此，让我们继续创建一个名为`Config.js`的文件，位于`config`目录中，并添加以下几行代码：

```js
module.exports = {
 DB: 'mongodb://localhost/movie_rating_app',
 SECRET: 'movieratingappsecretkey'
}
```

我们还添加了一个名为`/api/current_user`的`GET`路由来获取当前登录用户的信息。这个 API 使用一个名为`isLoggedIn`的中间件方法，检查用户数据是否在会话中。如果用户数据存在于会话中，则将当前用户详细信息作为响应发送回。

我们还添加了一个名为`/logout`的另一个端点，它简单地注销用户并销毁会话。

因此，有了这个配置，现在我们应该能够使用`passport.js`的 Local Strategy 成功登录。

我们现在唯一的问题是，没有方法知道用户是否成功登录。为此，我们需要显示一些用户信息，例如`email`，以指示已登录的用户。

为了做到这一点，我们需要从`Login.vue`传递用户信息到`App.vue`，这样我们就可以在顶部栏显示用户的电子邮件。我们可以使用`Vue`提供的名为`emit`的方法，该方法用于在`Vue`组件之间传递信息。让我们继续配置它。

# 配置 emit 方法

让我们首先创建一个可以用于不同 Vue 组件之间通信的传输器。在`src`目录内创建一个名为`bus.js`的文件，并添加以下内容：

```js
import Vue from 'vue';

const bus = new Vue();

export default bus;
```

现在，将 `Login.vue` 中 `script` 标签内的内容替换为以下代码：

```js
...
<script>
import axios from 'axios';
import bus from './../bus';

export default {
  data: () => ({
    valid: true,
    email: '',
    password: '',
    emailRules: [
      v => !!v || 'E-mail is required',
      v => /\S+@\S+\.\S+/.test(v) || 'E-mail must be valid',
    ],
  }),
  methods: {
    async submit() {
      return axios({
        method: 'post',
        data: {
          email: this.email,
          password: this.password,
        },
        url: 'http://localhost:8081/users/login',
        headers: {
          'Content-Type': 'application/json',
        },
      })
        .then(() => {
          this.$swal('Great!', 'You are ready to start!', 'success');
          bus.$emit('refreshUser');
          this.$router.push({ name: 'Home' });
        })
        .catch((error) => {
          const message = error.response.data.message;
          this.$swal('Oh oo!', `${message}`, 'error');
          this.$router.push({ name: 'Login' });
        });
    },
    clear() {
      this.$refs.form.reset();
    },
  },
};
</script>
```

在这里，我们正在发出一个名为 `refreshUser` 的方法，该方法将在 `App.vue` 中定义。将 `App.vue` 中的内容替换为以下代码：

```js
<template>
  <v-app id="inspire">
    <v-navigation-drawer
      fixed
      v-model="drawer"
      app
    >
      <v-list dense>
        <router-link v-bind:to="{ name: 'Home' }" class="side_bar_link">
          <v-list-tile>
            <v-list-tile-action>
              <v-icon>home</v-icon>
            </v-list-tile-action>
            <v-list-tile-content>Home</v-list-tile-content>
          </v-list-tile>
        </router-link>
        <router-link v-bind:to="{ name: 'Contact' }" class="side_bar_link">
          <v-list-tile>
            <v-list-tile-action>
              <v-icon>contact_mail</v-icon>
            </v-list-tile-action>
            <v-list-tile-content>Contact</v-list-tile-content>
          </v-list-tile>
        </router-link>
      </v-list>
    </v-navigation-drawer>
    <v-toolbar color="indigo" dark fixed app>
      <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
      <v-toolbar-title>Home</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-toolbar-items class="hidden-sm-and-down">
 <v-btn id="add_movie_link" flat v-bind:to="{ name: 'AddMovie' }"
 v-if="current_user">
 Add Movie
 </v-btn>
 <v-btn id="user_email" flat v-if="current_user">{{ current_user.email }}</v-btn>
 <v-btn flat v-bind:to="{ name: 'Register' }" v-if="!current_user" id="register_btn">
 Register
 </v-btn>
 <v-btn flat v-bind:to="{ name: 'Login' }" v-if="!current_user" id="login_btn">Login</v-btn>
 <v-btn id="logout_btn" flat v-if="current_user" @click="logout">Logout</v-btn>
 </v-toolbar-items>
    </v-toolbar>
    <v-content>
      <v-container fluid>
        <div id="app">
          <router-view/>
        </div>
      </v-container>
    </v-content>
    <v-footer color="indigo" app>
      <span class="white--text">&copy; 2018</span>
    </v-footer>
  </v-app>
</template>

<script>
import axios from 'axios';

import './assets/stylesheets/main.css';
import bus from './bus';

export default {
  data: () => ({
    drawer: null,
    current_user: null,
  }),
  props: {
    source: String,
  },
  mounted() {
 this.fetchUser();
 this.listenToEvents();
 },
  methods: {
    listenToEvents() {
 bus.$on('refreshUser', () => {
 this.fetchUser();
 });
 },
 async fetchUser() {
 return axios({
 method: 'get',
 url: '/api/current_user',
 })
 .then((response) => {
 this.current_user = response.data.current_user;
 })
 .catch(() => {
 });
 },
    logout() {
 return axios({
 method: 'get',
 url: '/api/logout',
 })
 .then(() => {
 bus.$emit('refreshUser');
 this.$router.push({ name: 'Home' });
 })
 .catch(() => {
 });
 },
  },
};
</script>
```

在这里，我们添加了名为 `refreshUser` 的方法，该方法在 `App.vue` 的 `mounted` 方法中被监听。每当用户登录到应用程序时，`App.vue` 中的 `refreshUser` 方法就会被调用，并获取已登录用户的信息。

此外，我们在顶部栏中显示了用户的电子邮件，这样我们就可以知道用户是否已登录。

此外，让我们也将 JWT 验证从电影控制器中移除。将 `controllers/movies.js` 中的内容替换为以下代码：

```js
const MovieSchema = require('../models/Movie.js');
const Rating = require('../models/Rating.js');

module.exports.controller = (app) => {
  // fetch all movies
  app.get('/movies', (req, res) => {
    MovieSchema.find({}, 'name description release_year genre', (error, movies) => {
      if (error) { console.log(error); }
      res.send({
        movies,
      });
    });
  });

  // fetch a single movie
  app.get('/api/movies/:id', (req, res) => {
    MovieSchema.findById(req.params.id, 'name description release_year genre', (error, movie) => {
      if (error) { console.error(error); }
      res.send(movie);
    });
  });

  // rate a movie
  app.post('/movies/rate/:id', (req, res) => {
    const newRating = new Rating({
      movie_id: req.params.id,
      user_id: req.body.user_id,
      rate: req.body.rate,
    });

    newRating.save((error, rating) => {
      if (error) { console.log(error); }
      res.send({
        movie_id: rating.movie_id,
        user_id: rating.user_id,
        rate: rating.rate,
      });
    });
  });

  // add a new movie
  app.post('/movies', (req, res) => {
    const newMovie = new MovieSchema({
      name: req.body.name,
      description: req.body.description,
      release_year: req.body.release_year,
      genre: req.body.genre,
    });

    newMovie.save((error, movie) => {
      if (error) { console.log(error); }
      res.send(movie);
    });
  });
};
```

通过这种方式，当用户登录到应用程序时，我们应该能够看到以下屏幕：

![](img/2a7fa817-15de-4d8c-acf8-5d78928920ff.png)

# 摘要

在本章中，我们介绍了 `passport.js` 的工作原理。我们还介绍了如何在 MEVN 应用程序中使用简单的 JWT 策略，并处理用户的注册和登录。

在下一章中，我们将深入探讨不同的 `passport.js` 策略，例如 Facebook 策略、Google 策略和 Twitter 策略。
