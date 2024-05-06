# tniy-url-backend
NodeJS module to create tiny url (nanoid) for long url mapped to a unique counter.
Users are redirected to the original URL when they hit these short links.

Live Deployment Info

1.Frontend is deployed with Netlify
https://tinyurltest.netlify.app/

2.Backend is deployed with Heroku
https://secure-mesa-82846-88fa7338d24a.herokuapp.com/



#### Technologies used
    - NodeJS
    - MongoDB
	- Nanoid
	- Heroku
	- Netlify


## Usage
+ Url to test for backend page deployment (test only)

Example: https://secure-mesa-82846-88fa7338d24a.herokuapp.com/

Response:{"message":"Home page"}

```javascript
app.get("/", (req, res) => {
  res.json({
    message: "Home page",
  });
});
```

+ Url to get all stored data from database collection (test only)

Example: https://secure-mesa-82846-88fa7338d24a.herokuapp.com/urls

Response:[
  {
    "_id": "6637f93d36d45b360033a73d",
    "originalUrl": "https://www.geeksforgeeks.org/how-to-deploy-mongodb-app-to-heroku/",
    "slug": "679dc033",
    "createdAt": "2024-05-05T21:25:17.016Z",
    "updatedAt": "2024-05-05T21:25:17.016Z",
    "__v": 0
  },
  {
    "_id": "6637fa2a36d45b360033a73e",
    "originalUrl": "https://blog.onlineshoppingtools.com/tb/pricing?popup=1&utm_source=n&utm_medium=atn&atnid=175fddea-ca1b-44aa-b8fa-8ef2bf0c515a&uxid=tb&atnds=18&ec=0&uxid0=3939351962#tblciGiAM4jQK2PYwiJtG6J2h-NgmP7ZDCwx7OT_D77_B7Gg3GyD-_lMouITQn_r9gZ6-AQ",
    "slug": "8602e11f",
    "createdAt": "2024-05-05T21:29:14.978Z",
    "updatedAt": "2024-05-05T21:29:14.978Z",
    "__v": 0
  }
]

```javascript
app.get("/urls", async (req, res, next) => {
  let urls = await URL.find({}).exec();
  res.json(urls);
});
```

+ Shorten URL

Example: https://secure-mesa-82846-88fa7338d24a.herokuapp.com/api/shorten

This url is used to shorten long url to a "slug" url, i.e. short url(because later it would be appended back for redirect the url),First, the given url is been looked up in database to see if it had been created before, if yes, then the short url will be given again to user. If not, the validation of given url would be verified by axios request, if the http/https request is valid, then by using nanoid module's hashing mechanism, the slug(short url) would be created.

```javascript
app.post("/api/shorten", async (req, res, next) => {
  if (req.body.url) {
    try {
      let url = await URL.findOne({ originalUrl: req.body.url }).exec();

      if (url) {
        res.json({ short: `${process.env.URL}/${url.slug}`, status: 200 });
      } else {
        // make a request with Axios
        const response = await axios.get(req.body.url.toString(), {
          validateStatus: (status) => {
            return status < 500;
          },
        });

        if (response.status != 404) {
          let newUrl;
          while (true) {
            let slug = nanoid();
            let checkedSlug = await URL.findOne({ slug: slug }).exec();
            if (!checkedSlug) {
              newUrl = await URL.create({
                originalUrl: req.body.url,
                slug: slug,
              });
              break;
            }
          }

          res.json({
            short: `${process.env.URL}/${newUrl.slug}`,
            status: response.status,
          });
        } else {
          res.json({
            message: response.statusText,
            status: response.status,
          });
        }
      }
    } catch (err) {
      next(err);
    }
  } else {
    res.status(400);
    const error = new Error("URL is required");
    next(error);
  }
});
```

+ Retrieve Long URL from hash

This url is used for redirecting to originalUrl for user: 

Example: https://secure-mesa-82846-88fa7338d24a.herokuapp.com/1e3ac11d

```javascript
app.get("/:slug", async (req, res, next) => {
  try {
    let url = await URL.findOne({ slug: req.params.slug }).exec();

    if (url) {
      res.status(301);
      res.redirect(url.originalUrl);
    } else {
      next();
    }
  } catch (err) {
    next(err);
  }
});
```

+ Simple Error handling

This url is used for simple error handling on invalid url usage on backend, "Not found" msg will be reponse to user.
Example: https://secure-mesa-82846-88fa7338d24a.herokuapp.com/INValidurl

Response: {"message":"Not found - /INValidurl","error":{"status":404}}

```javascript
function notFound(req, res, next) {
  res.status(404);
  const error = new Error("Not found - " + req.originalUrl);
  next(error);
}
```
## Data Schema(MongoDB)
```javascript
let UrlSchema = new mongoose.Schema(
  {
    originalUrl: {
      type: String,
      required: true,
    },
    slug: {
      type: String,
      required: true,
    },
  },
  { timestamps: true }
);
```