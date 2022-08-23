source: https://mfikri.com/artikel/login-jwt-express-react-mysql

# Backend-JWT-Node-Express-MYSQL
Backend JWT Node Express MYSQL

Step #1. Backend
#1.1. Install Dependency
Buat sebuah folder di komputer Anda, di sini saya beri nama “jwt-auth”.

Anda bebas membuatnya di manapun, baik di C, D, ataupun di Desktop.

Kemudian buka folder “jwt-auth” tersebut menggunakan code editor, disini saya menggunakan Visual Studio Code.

Saya juga menyarankan Anda untuk menggunakan Visual Studio Code.

Anda dapat mendownload Visual Studio Code pada link berikut, kemudian instal di komputer Anda:

https://code.visualstudio.com/

Setelah folder “jwt-auth” ter-open menggunakan Visual Studio Code, buat sebuah sub folder bernama “backend” di dalam folder “jwt-auth”.

Selanjutnya, buka terminal pada Visual Studio Code pada menu bar terminal => new terminal.

Setelah itu, ketikkan perintah berikut pada terminal untuk membuat file “package.json”:

1
npm init -y
Selanjutnya, install express, mysql2, sequelize, jsonwebtoken, bcrypt, cookie-parser, dotenv dan cors dengan mengetikan perintah berikut pada terminal:

1
npm install express mysql2 sequelize jsonwebtoken bcrypt cookie-parser dotenv cors
Selanjutnya, install nodemon secara global dengan mengetikan perintah berikut pada terminal:

1
npm install -g nodemon
Selanjutnya, tambahkan kode berikut pada file “package.json”:

1
"type": "module",
Sehingga file “package.json” terlihat menjadi seperti berikut:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
{
  "name": "backend",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "test": "echo "Error: no test specified" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "bcrypt": "^5.0.1",
    "cookie-parser": "^1.4.5",
    "cors": "^2.8.5",
    "dotenv": "^10.0.0",
    "express": "^4.17.1",
    "jsonwebtoken": "^8.5.1",
    "mysql2": "^2.3.2",
    "sequelize": "^6.8.0"
  }
}
Hal ini bertujuan agar kita dapat menggunakan ES6 Module Syntax untuk export dan import module.

Dapatkan diskon 75% paket hosting dan gratis domain + extra diskon 5% dengan menggunakan kupon: MFIKRI
ORDER SEKARANG.!
 

#1.2. Buat Database
Untuk dapat menggunakan MySQL, Anda perlu menginstall XAMPP, WAMP, MAMP, atau software sejenisnya.

Pada tutorial ini, saya menggunakan XAMPP.

Kemudian buat database baru pada MySQL, Anda dapat menggunakan tools seperti SQLyog, PHPMyAdmin atau sejenisnya.

Disini saya membuat database dengan nama “auth_db”.

Jika Anda membuat database dengan nama yang sama, itu lebih baik.

Untuk  membuat database pada MySQL, dapat dilakukan dengan mengeksekusi query berikut:

1
CREATE DATABASE auth_db;
Perintah SQL diatas akan membuat sebuah database dengan nama “auth_db”.

 

#1.3. Struktur Aplikasi
Agar aplikasi lebih terstruktur rapi, kita akan menerapkan pola MVC (Model-View-Controllers).

Buat folder “config”, “controllers”, “middleware”, “models”, dan “routes” di dalam folder “backend”.

Kemudian buat file “Database.js” di dalam folder “config”, buat file “Users.js” dan “RefreshToken.js” di dalam folder “controllers”, buat file “VerifyToken.js” di dalam folder “middleware”, buat file “UserModel.js” di dalam folder “models”, buat file “index.js” di dalam folder “routes”, dan buat file “index.js” di dalam folder “backend”.

Perhatikan gambar berikut untuk lebih jelasnya:

struktur project

 

#1.4. Connect ke Database
Buka file “Database.js” yang terdapat pada folder “config”, kemudian ketikan kode berikut:

1
2
3
4
5
6
7
8
import { Sequelize } from "sequelize";
 
const db = new Sequelize('auth_db', 'root', '', {
    host: "localhost",
    dialect: "mysql"
});
 
export default db;
 

#1.5. Models
Buka file model “UserModel.js” yang terdapat pada folder “models”, kemudian ketikan kode berikut:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
import { Sequelize } from "sequelize";
import db from "../config/Database.js";
 
const { DataTypes } = Sequelize;
 
const Users = db.define('users',{
    name:{
        type: DataTypes.STRING
    },
    email:{
        type: DataTypes.STRING
    },
    password:{
        type: DataTypes.STRING
    },
    refresh_token:{
        type: DataTypes.TEXT
    }
},{
    freezeTableName:true
});
 
(async () => {
    await db.sync();
})();
 
export default Users;
 

#1.6. Controllers
Buka file controller “Users.js” yang terdapat pada folder “controllers”, kemudian ketikan kode berikut:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
import Users from "../models/UserModel.js";
import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";
 
export const getUsers = async(req, res) => {
    try {
        const users = await Users.findAll({
            attributes:['id','name','email']
        });
        res.json(users);
    } catch (error) {
        console.log(error);
    }
}
 
export const Register = async(req, res) => {
    const { name, email, password, confPassword } = req.body;
    if(password !== confPassword) return res.status(400).json({msg: "Password dan Confirm Password tidak cocok"});
    const salt = await bcrypt.genSalt();
    const hashPassword = await bcrypt.hash(password, salt);
    try {
        await Users.create({
            name: name,
            email: email,
            password: hashPassword
        });
        res.json({msg: "Register Berhasil"});
    } catch (error) {
        console.log(error);
    }
}
 
export const Login = async(req, res) => {
    try {
        const user = await Users.findAll({
            where:{
                email: req.body.email
            }
        });
        const match = await bcrypt.compare(req.body.password, user[0].password);
        if(!match) return res.status(400).json({msg: "Wrong Password"});
        const userId = user[0].id;
        const name = user[0].name;
        const email = user[0].email;
        const accessToken = jwt.sign({userId, name, email}, process.env.ACCESS_TOKEN_SECRET,{
            expiresIn: '20s'
        });
        const refreshToken = jwt.sign({userId, name, email}, process.env.REFRESH_TOKEN_SECRET,{
            expiresIn: '1d'
        });
        await Users.update({refresh_token: refreshToken},{
            where:{
                id: userId
            }
        });
        res.cookie('refreshToken', refreshToken,{
            httpOnly: true,
            maxAge: 24 * 60 * 60 * 1000
        });
        res.json({ accessToken });
    } catch (error) {
        res.status(404).json({msg:"Email tidak ditemukan"});
    }
}
 
export const Logout = async(req, res) => {
    const refreshToken = req.cookies.refreshToken;
    if(!refreshToken) return res.sendStatus(204);
    const user = await Users.findAll({
        where:{
            refresh_token: refreshToken
        }
    });
    if(!user[0]) return res.sendStatus(204);
    const userId = user[0].id;
    await Users.update({refresh_token: null},{
        where:{
            id: userId
        }
    });
    res.clearCookie('refreshToken');
    return res.sendStatus(200);
}
Setelah itu, buka file controller “RefreshToken.js” yang terdapat pada folder “controllers”, kemudian ketikan kode berikut:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
import Users from "../models/UserModel.js";
import jwt from "jsonwebtoken";
 
export const refreshToken = async(req, res) => {
    try {
        const refreshToken = req.cookies.refreshToken;
        if(!refreshToken) return res.sendStatus(401);
        const user = await Users.findAll({
            where:{
                refresh_token: refreshToken
            }
        });
        if(!user[0]) return res.sendStatus(403);
        jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET, (err, decoded) => {
            if(err) return res.sendStatus(403);
            const userId = user[0].id;
            const name = user[0].name;
            const email = user[0].email;
            const accessToken = jwt.sign({userId, name, email}, process.env.ACCESS_TOKEN_SECRET,{
                expiresIn: '15s'
            });
            res.json({ accessToken });
        });
    } catch (error) {
        console.log(error);
    }
}
 

#1.7. Middleware
Buka file “VerifyToken.js” yang terdapat pada folder “middleware”, kemudian ketikan kode berikut:

1
2
3
4
5
6
7
8
9
10
11
12
import jwt from "jsonwebtoken";
 
export const verifyToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    if(token == null) return res.sendStatus(401);
    jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, decoded) => {
        if(err) return res.sendStatus(403);
        req.email = decoded.email;
        next();
    })
}
 

#1.8. Routes
Buka file “index.js” yang terdapat pada folder “routes”, kemudian ketikan kode berikut:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
import express from "express";
import { getUsers, Register, Login, Logout } from "../controllers/Users.js";
import { verifyToken } from "../middleware/VerifyToken.js";
import { refreshToken } from "../controllers/RefreshToken.js";
 
const router = express.Router();
 
router.get('/users', verifyToken, getUsers);
router.post('/users', Register);
router.post('/login', Login);
router.get('/token', refreshToken);
router.delete('/logout', Logout);
 
export default router;
 

#1.9. Entry Point
Buka file “index.js” yang terdapat pada folder “backend”, kemudian ketikan kode berikut:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
import express from "express";
import dotenv from "dotenv";
import cookieParser from "cookie-parser";
import cors from "cors";
import db from "./config/Database.js";
import router from "./routes/index.js";
dotenv.config();
const app = express();
 
app.use(cors({ credentials:true, origin:'http://localhost:3000' }));
app.use(cookieParser());
app.use(express.json());
app.use(router);
 
app.listen(5000, ()=> console.log('Server running at port 5000'));
 

#1.10. .env
Buat sebuah file bernama “.env” di dalam folder “backend”, kemudian ketikan kode berikut:

1
2
ACCESS_TOKEN_SECRET = jsfgfjguwrg8783wgbjs849h2fu3cnsvh8wyr8fhwfvi2g225
REFRESH_TOKEN_SECRET = 825y8i3hnfjmsbv7gwajbl7fobqrjfvbs7gbfj2q3bgh8f42
Anda bebas memasukan value pada ACCESS_TOKEN_SECRET dan REFRESH_TOKEN_SECRET diatas.

Untuk memastikan aplikasi berjalan dengan baik, jalankan aplikasi dengan mengetikan perintah berikut pada terminal:

1
nodemon index
Jika berjalan dengan baik, maka akan terlihat seperti gambar berikut:

nodemon

Sampai disini Anda telah berhasil membuat “backend”.
