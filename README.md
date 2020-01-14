# Passport
## [**Passport**](http://(http://www.passportjs.org/))란?
    인증을 복잡한 로직없이 편하게 작업할 수 있게 도와주는 Node.js 미들웨어
    OAuth인증, username/password을 통한 인증등의 복잡한 코딩없이 passport에서 지원하는 method를 사용하면 간결.
    NodeJS서버측에서 세션을 관리하며, 유저의 요청에 세션ID를 부여해 유저에게 쿠키형태로 준다
    어느 유저가 어느 쿠키를 가지고있는지 식별해줄 것이며 세션/쿠키의 원리로 움직인다.
    passport-local, passport-github등의 자식을 가진다.

## 0. 쉬운 이해를 위한 프리뷰 
--------------------------------
<br>

* HTTP통신 <br>
  비연결지향의 HTTP통신은 연결을 유지시켜주는 것이 아닌 연결이 되고나면 연결을 끊어버린다.
  이러한 특성에도 불구하고 우리의 NAVER페이지는 어떻게 로그인이 된 상태로 유지되어 있는걸까?<br>
<br>

* 쿠키<br>
유저의 브라우저에 저장되는 텍스트 형식의 데이터
쿠키에 계정 정보 (ex ID, Password ...)를 담아서 서버로 전송하여 인증하는 방식이다. 
  페이지 이동시마다 사용자 정보를 보내게 되는데, 
  누가 HTTP 요청을 확인한다면 사용자의 계정 정보가 그대로 노출이 되기 때문에 보안성이 없다
<br>  
<br>  
* 세션  
  쿠키가 유저의 브라우저에 저장되는 데이터였다면, 세션은 서버에 저장되는 데이터를 말한다.
<br>
<br>  

* Serialize<br> 
쿠키에 어떤 정보를 넣을 것인지에 대한 작업<br>
놀이공원 매표소에서 입장권을 끈는 과정으로 예를 든다.    
신분증을 보여주고 대한민국 국민임이며 김영민이라는 사람으로 3시간 자유이용권을 끈는다.<br>    <br>

        특정 사이트에 로그인을 성공하였고, 로그인 정보를 parsing해 서버측의 session에 저장한다.    
        김영민+3시간자유이용권 이라는 정보가 담긴 팔찌(쿠키)를 부여한다
        session Body에는 김영민에 대한 각종정보+3시간 자유이용권이 들어가 있고
        sessonID에는 김영민이란 사람이 자유이용권을 가지고 있다는 정보가 담겨져있다.
        놀이공원 직원은 이 팔찌를 보고 내가 어떤 사람인지 판별할 수 있고, 입장시켜 준다.
        sessionID라는 팔찌를 받고 놀이기구를 탈 때마다 인증을 하지않아도 들어갈 수 있다.

<br><br>


* Deserializiation<Br>
    팔찌에 자유이용권 이용자의 정보가 담겨져 있고, 직원들은 팔찌만 보고 입장을 허용한다.<br> 
    페이지 이동시(놀이기구 입장시) deserialze를 호출하고, 입장권을 확인 해 '김영민님이시네요! 들어가시죠' 

  
<br>

  * OAuth<br>
  유저들이 비밀번호를 제공하지 않고 다른 사이트 계정의 정보를 받아 접근권한을 얻는 인증 방식을 말한다. <br>
  깃헙으로 예를 들어보자<br>
    YTube유저 - 깃헙으로 로그인 -> 깃헙사이트 -> 깃헙정보를 제공하는 것에 동의 -> 깃헙의 callbackUrl(로그인 로직처리) <br>  -> YTube에서 github계정을 가져와 회원가입/로그인 성공!
    <br>
  <br><br><br>
    

## 1. Passport의 대략적인 흐름
--------------------------------------
### 1.1 미들웨어 (`실행,사용`이라기 보단 `선언`이라는 느낌으로 받아들인다)<br>
---------------------------------
[Passport Configure](http://www.passportjs.org/docs/configure/) 참조

    
```
    app.use(session)                //세션정보를 전송할 쿠키에 대한 여러가지 설정값들
    app.use(passport.initialize)    //req.user이라는 객체로 passport.auth의 전략대로 정보가 들어간다
    app.use(passport.session)       //passport를 사용해 인증한 유저에 대해 sessionID값을 부여
```
<br>

   ### passport.session VS app.session

    
    1. express-session
    app.use(session ({ 쿠키의 보안성/쿠키의 경로/ 쿠키의 생존시간/ 쿠키의 도메인 등등 }) )
    유저에게 줄 세션ID(쿠키)의 생존시간/ 쿠기가 저장되는 경로/ 보안성 등등에 대한 설정을 위한 미들웨어.
    자유이용권의 처리방식에 대해 다루는 녀석
  
    2. passport.session
    서버는 session을 가지고 있고 유저는 그에 상응하는 session ID를 쿠키의 형태로 가진다. 
    passport.session은 passport의 전략을 가지고 인증(local/ github)을 했을 때 "sessionID"를 주는 녀석.
    자유이용권을 팔찌의 형태로 만들어서 주는 것과 일맥상통하다. 

    그리고, 자유이용권의 처리방식에 대해선 app.session을 따른다
        = 세션의 유효기관 이나 세션에대한 설정은 app.session을 따른다.

    app.session이 passport.session보다 거대하며 passport.session을 품고있다.
    


<br><br>

### 1.2 passport설정부분(passport.js)
----------------------------

```
   passport.use(전략)     //passport내의 전략을 사용
   passport(serialize)    //passport 전략에 따른 sessionID부여하는 부분. 자유이용권 팔찌 팔에 둘러주는 부분
   passport(deserialize)  //자유이용권 유저를 확인/판별하는 코드   
```
<br>

  ### 1.3 passpoar의 전체적인 흐름도
  ---------------------------------

    1.로그인요청 //get.post( url, func )
    2.func가 passport관련 함수라면 
        2-1)passport.js로 와서 대응되는 전략/전략설정을 토대로 전략을 실행
        2-2)passport내장함수로 로그인이 성공하면 serialize로 팔찌를 얻고 deserialize로 페이지이동시마다 확인하는       과정이 실행된다
    3.자유이용권 팔찌들은 express-session의 설정을 따르며, passport초기화, passport녀석들에게 sessionID를 쓰는 정책을 쓴다
  
<br>

## 2. Passport 전략
-------------------------  
  ### 1. passport-local

passport에는 많은 전략이 있지만 username/password로 인증을 거치는 `전략`.<br>
여기서 username은 id가 될 수도 있고 email등 여러가지가 될 수 있다.<br>
DB 스키마와 플러그인을 맺어줄 때 설정할 수 있다. 이는, passport-local-mongoose의 메소드로 설정할 수 있다.

<br>

### 1.1) [passport-local-mongoose](https://github.com/saintedlama/passport-local-mongoose#api-documentation)
passport-local만으로도 로그인 구현이 가능하지만, 더 간단하게 가능하게 해준다.<br>
passport-local의 핵심전략 username/password를 통한 인증의 방식으로, username option을 설정할 수 있다.<br>

    ex) UserSchema.plugin(passportLocalMongoose, { usernameField: "email" });<br>



<br><br>

# 3. Passport 사용하기

## 3-1) passport-local(feat.mongoose)
----------------------------  
### 1. install
------------------------------
>passport는 세션원리이기 때문에 passport, passport-local, passport-local-mongoose, express-session
>위의 4가지를 다운받는다.
    
        npm install passport passport-local passport-local-mongoose express-session
    
    
###   2.  plugin
------------------------------
DB User스키마와 passport-local-mongoose와 연결.(물론 passport-local만으로도 구현가능하지만 복잡하다)
      
  
####  **[ User.js ]**
```
  import mongoose from "mongoose";
  import passportLocalMongoose from "passport-local-mongoose";

  const UserSchema = new mongoose.Schema({ 아무렇게나 만들고 });          
                                            //스키마생성
  
  UserSchema.plugin(passportLocalMongoose, { usernameField: "email" });   
                                            //email과 password를 비교해서 로그인한다.

  const model = mongoose.model("User", UserSchema);
  export default model;
```
<br>
  
### 3.  passport전략 환경설정
------------------------------

  #### **[ passport.js ]**
  

  ```
  import passport from "passport";
  import LocalStrategy from "passport-local";
  import User from "./models/user";


  /*
    passport.use(new LocalStrategy(User.authenticate()));
    나는 이 방법이 new도 있고 다른 passport configure와 비슷해서 이거 사용했었는데
    new LocalStrategy가 먹히는 건 namefiled option에 손 안댔을 때만, 우리는 email로 바꿨으므로
    User.createStrategy()를 써야한다.
  */
  
  passport.use(User.createStrategy());
  passport.serializeUser(User.serializeUser());
  passport.deserializeUser(User.deserializeUser());
  
  //NodeJS가 세션관리하고 유저에게 보내고 유저의 요청으로부터 세션을 해독할 때 쓰이는 serialize들.
```
<br>

### 4. passport/ session 선언
------------------------------
  
  #### **[ app.js ]**
  ```
  import session from "express-session";
  import passport from "passport";
  import cookieParser from "cookie-parser";
  import "./passport";
  
  app.use(
  session({
    secret: "keyboard cat",
    cookie: { maxAge: 60 * 60 * 1000 },
    resave: true,
    saveUninitialized: false,
    store: new CookieStore({ mongooseConnection: mongoose.connection }) 
        //쿠키 Store의 가장 중요한 부분. 몽고와 연결해줘야 몽고 저장소에 저장가능함.
    })
  );
  app.use(passport.initialize());
  app.use(passport.session());
  ```

<br>

### 5. 하면좋고 안해도 전혀 문제X
------------------------------
서버가 종료되면 유저의 정보들은 사라지게 된다. <br>
웹 사이트 업데이트를 한다고 하고 완료되면 유저의 정보들은 사라지기 때문에 유저입장에서는 불편할 수 있다.<br>
서버가 재시작되어도 유저정보를 저장할 수 있게 만드는 것을 만들어보자
      
원리는 세션/쿠키를 저장할 또 다른 mongoDB의 어느 저장소에 저장을 하는 방식.

    npm i connect-mongo
    
session을 Server에 저장도 하고, DB에 저장해서 Server가 업데이트 되어도 DB의 세션정보를 불러온다.<br>
Server의 세션을 DB에 저장할 수 있게 도와주는 미들웨어.

<br>

#### **[ app.js ]**
```      
import MongoStore from "mongo-connect"; 
      
const CookieStore = MongoStore(session);
app.use(session({
    store: new CookieStore({ mongooseConnection: mongoose.connection })  
        //session의 저장경로를 mongoose의 CookieStore로 설정한다.
    })
);     
```     
        
        
<br>

  ### 6. 회원가입처리 로직 (이 부분은 MongoDB 내용이라 passport와 상관은 없음. video업로드와 상당히 유사)
  ------------------------------
  #### **[ userController.js ]**
               
```
  export const postJoin = async (req, res, next) => {
  /*
    user의 post요청 - - bodyParser(middleware) - - router - - controller - - redirect("join")
    미들웨어의 순서는 app.js에 선언해놓은 순서대로! router제일 마지막이고 controller/pug는 end point!
    bodyParser가 있을 경우 req.body.[pug의 name="이 값"]으로 취할 수 있다
  */

  const name = req.body.name;
  const email = req.body.email;
  const password = req.body.password;
  const password2 = req.body.password2;

  if (password !== password2) {
    res.status(400);
  } else {
    try {
      const user = {
        name,
        email
      };
      await User.register(user, password);
      next();
    } catch (error) {
      console.log(error);
      res.redirect(routes.join);
    }
  }
};
```
    await User.register(user, password);
User.js의 스키마에는 Password라는 속성이 없다. 이는 보안을 위해서라고 생각되어진다.<br>
하지만, password 속성이 없는데 passport-local은 어떻게 username(email)과 password를 비교해서 로그인이 가능한 걸까?
User.register()라는 메서드는 passport-local-mongoose의 메서드이고, User.create보다 더 안전적이다.<br>
User.register로 유저는 user정보와 password를 가지고 회원가입을 했으며, 비밀번호를 변경할 때는 passport-local-mongoose의 req.user.changePassword(old ,new)라는 메서드로 변경할 수 있다.

<br><br>

  ### 6. 가입 후 자동 로그인처리
  -----------------------
#### 1.  미들웨어순서
>express-session > passport.initialize > passport.session > routes > controller > passport인증전략 > de/serialize > webPage

<br>

#### 2. Router순서 (회원가입 후 로그인 처리)<br>
join post방식 > postJoinController(회원가입로직) > postLoginController(로그인로직) > homePage(홈페이지)

<br>
  
#### [ globalRouter.js ]
    globalRouter.post(routes.join, postJoin, postLogin);
            
#### [ userController.js ]
```
    export const postLogin = (req, res) => {
      passport.authenticate(`local`,
      {   
        failureRedirect: routes.login,      
        successRedirect: routes.home        
      })
    };    
```
<br>

     

#### 3.  passport.authenticate
    passport.authenticate(`local`, ~ )
passport.authenticate( 전략종류 )<br>
여기서는 passport-local방식이므로 `local`이 들어가고, passport.authenticate메서드가 호출되면 전략방식을 따르며,<br>
[ passport.js ] 에서 설정한 전략설정/방식대로 메서드가 실행된다. <br>
return이 참이면 해당Redirect로 이동.<br>
현재로써는 설정된 username(email)과 pass찾아보도록 설정되어있다.




## 3-2) passport-github
----------------------------  
### 1. install
------------------------------
    npm install passport-github

<Br>


### 2.  preview
---------------------
  
###  1) "깃헙 개발자 사이트"에서 내 웹사이트 등록
    

    YTube: "github으로 계속하기" 기능을 써서 내 로그인시스템을 구축하려고 해. 
            너네 user들 좀 데려다가 쓸 수 있을까?

    Github: YTube? 그게뭐야? 우리 유저끌어다 쓰고싶으면 너네 웹사이트를 Github에 등록해. 
            등록안하면 네가 누군지 모르고  무슨사이트인지도 모르기 때문에 우리 유저정보 함부로 줄 수 없어. 
            등록하면 유저정보 허용하게 해줄께.  https://github.com/settings/applications/new
    YTube:
        -Application name : YTube
        -Homepage URL : localhost:4000
        -callbackUR: : /auth/github/callback
        
        이 정보들이 github DB에 들어가고, YTube의 유저가 "github으로 계속"을 클릭하면
        url과 name으로 내 웹사이트를 찾고 YTube는 등록이 되어있기 때문에 user정보를 YTube로 넘겨줄 수 있다.
        그리고 callbackURL은 "김님으로 계속하시겠습니까"를 클릭하면 이동되는 url이다. 
        라우팅으로 따로 표출되어지는 웹 페이지는 없지만 이곳에서 깃헙의 각종로그인 로직들이 실행되어진다
        github의 유저정보는 profile에 저장되며, 각종 토큰/세션과 콜백을 보내는 함수로 YTube로 보내준다.
        위의 과정으로 github은 YTube로부터의 github로그인 요청이 들어오면 정보들을 제공할 준비가 완료되었다.

<br><br>

### 2) passport-github "전략설정/수립"   

#### [ passport.js ]
```
    import passport from "passport";
    import GitHubStrategy from "passport-github";
    
    passport.use(
      new GitHubStrategy(
      {
        clientID: process.env.GITHUB_CLIENT_ID,
        clientSecret: process.env.GITHUB_CLIENT_SECRET,
        callbackURL: `http://localhost:4000${routes.githubCallback}`
      },
      githubCallback

/*  
    callbackURL에서 일어나는 일들이 담겨져있는 함수. 파라미터로 token,requestToken,profile,cb가 있다
    = 김님으로 계속을 클릭하면 벌어지는 함수.
    인증을 받고 사용자가 깃헙으로부터 우리YTube로 돌아왔을 떄 github의 유저정보/토큰 등을 받는 함수
*/
  )
);
```
<Br>

passport.use()
```
    passport.use(
      new GitHubStrategy(
      {
        clientID: process.env.GITHUB_CLIENT_ID,
        clientSecret: process.env.GITHUB_CLIENT_SECRET,
        callbackURL: `http://localhost:4000${routes.githubCallback}`
      },
```

>clientID와 clientSecret은 github 개발자 등록완료하면 부여받는 ID와 SECRET값이다.<br>
>타인이 보면 좋지 않기 때문에 .env에 기입해두고 가져와서 쓰도록하자.<br>

>그리고 callbackURL은 "김님으로 계속하시겠습니까?"의 과정에서 확인을 누르면 이동되어지는 url이고<br>
>이 url은 일반적으로 github/auth/callback와 같이 설정한다.


<br>

  ### githubCallback정의
1. github에서 받은 정보들을 토대로 YTube의 User model에 회원정보가 중복되는 지 확인한다<br> 
2. 그렇지 않다면 받은 정보를 토대로 회원가입실시<br>
3. github의 이름, 이메일, id 등등 가지고와서 YTube User테이블에 넣을것이다.
      

<br>

  #### [ userController.js ]
``` 
  export const githubCallback = async ( accessToken, refreshToken, profile, cb ) => {

    const id = profile._json.id;
    const avatarUrl = profile._json.avatar_url;
    const name = profile._json.name;
    const email = profile._json.email;

    try {
      const user = await User.findOne({ email:email });   //email로 유저찾기
      if (user) {
        user.githubId = id;   
        user.save();
          /*
            위의 2줄은 빼도 무방하지만, 이런 경우일 때를 대비해서 코드를 작성하였다.
            이미 caca4616@naver.com로 local회원가입한 유저가 있는 상황
                + github으로 회원가입을 했는 데 github email도 caca4616@naver.com인 경우
                email로 찾기 때문에 기존의 유저에게 대응되며, 그때는 githubId를 업데이트 하는 형식.
                기존의 local방식으로 회원가입 하지 않았다면 코드가 없든 있든 전혀 상관없는 문제.
          */
        return cb(null, user);
      } else {
        const newUser = await User.create({
          email,
          name,
          githubId: id,
          avatarUrl
        });
        return cb(null, newUser);
      }
    } catch (error) {
      return cb(error);
    }
  };
```

  이 callback함수에서는 `로그인을 실시한다`라는 로직은 없었다. 오직, 유저등록뿐이었다.
  내 생각이지만, 로그인하는 로직은 passport.authenticate(`github`)여기에 있는 게 아닐까 생각한다.
    
<br><br>           
              
###  3.Github인증페이지로 "라우팅"
-------------------------
YTube유저가 github로그인 클릭했을 때, github인증페이지로 이동하는 url과 그 페이지에서 벌어지는 함수를 작성한다.
<Br>

#### [ globalRouter.js ]
    import passport from "passport";
    
      app.get('/auth/github', passport.authenticate('github'));
      app.get('/auth/github/callback', 
              passport.authenticate('github', { failureRedirect: '/login' }),
              function(req, res) {
                res.redirect('/');
      });

>1번째의 /auth/github에 대한 요청이 들어오면, passport.authenticate(`github`)이라는 함수를 발견하면 <br>
[ paasport.js ]로 가서 전략설정값들을 확인하고 github전략 로직(passport내장)을 실행한 후 /auth/github으로 이동한다.

      
>2번쨰의 passport.authenticate(`github`, { } )은 `김님으로 계속`을 클릭하면 실행되는 녀석들이다.
      callback url은 우리가 볼 수 없는 숨겨진 url이지만, callback으로 가면, github내에서 내부적으로 함수들이 실행된다.<br>
      [passport.js]에서 불러왔던 githubCallback함수도 실행이 되고, 모든 함수들이 정상적으로 작동했다면 redirect(`/`)로 돌아온다.
      
      