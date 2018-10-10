Title: Zooming Past the Competition
Slug: taboola-hackathon-2018
Date: 2018-10-10 21:00
Tags: deep learning, web, hackathon, augmented reality
Summary: How to create an Augmented Reality app that allows a user to get content recommendations.
header_cover: images/hackathon_2018/cover.png

Imagine you’re walking down the street and you see a nice car you’re thinking of
buying. Just by pointing your phone camera, you can see relevant content about
that car. How cool is that?!

That was our team’s idea that awarded us first place in the recent Taboola R&D
hackathon aptly named — [Taboola Zoom](https://zoom.taboola.com/)!

Every year Taboola holds a global R&D hackathon for its 350+ engineers aimed at
creating ideas for cool potential products or just some fun experiments in
general.

This year, 33 teams worked for 36 hours to come up with ideas that are both
awesome and helpful to Taboola. Some of the highlights included a tool that can
accurately predict the users’ gender based on their browsing activity and an
integration to social networks for Taboola Feed.

Our team decided to create an
[AR](https://en.wikipedia.org/wiki/Augmented_reality) (Augmented Reality)
application that allows a user to get content recommendations, much like
Taboola’s recommendations widget, based on whatever they’re pointing their phone
camera at.

# What is Zoom?
![](images/hackathon_2018/demo.png)

The app itself is an AR experience similar to that of Google Glass, which allows
you to interact with the world using your phone camera. Using the app, one just
needs to point their camera onto an object to immediately get a list of stories
from the web related to that object.

To make this idea a reality we used technologies from several domains —
[AI](https://en.wikipedia.org/wiki/Artificial_intelligence), [Web
Applications](https://en.wikipedia.org/wiki/Web_application) and
[Microservices](https://en.wikipedia.org/wiki/Microservices).

### Under the Hood
![](images/hackathon_2018/architecture.png)

The flow is pretty simple:

1.  The user opens a web application on his phone — an
[HTML5](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5) page that
behaves like a native app.
2.  The app sends a screenshot of the captured video every second to a remote
server.
3.  The server processes the image using computer vision technologies.
4.  The server searches for web articles with thumbnails that are the most similar
to the processed image.
5.  The retrieved images are filtered using a similarity threshold, and are sent
back to the user to be shown in a slick widget atop of the user’s display.
6.  When the user clicks the widget the relevant article is opened in the browser.

### Zooming In
Each component of the system is implemented using different technologies:

#### Web Application
We decided to implement the user interface using HTML5, which allowed us access
to native capabilities of the phone such as the camera. Additionally, we used
[WebRTC API](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API) and
[Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API).

#### Computer Vision Service
![](images/hackathon_2018/vision.jpg)

For every image, the service processes the image and returns an
[embedding](https://developers.google.com/machine-learning/crash-course/embeddings/video-lecture)
— a numerical vector representing information extracted from the content of the
image.

Understanding the content of an image is a well known problem with plenty of
solutions. We chose to use Google’s
[Inception](https://research.googleblog.com/2016/03/train-your-own-image-classifier-with.html)
model, which is a [DNN](https://en.wikipedia.org/wiki/Deep_learning) (Deep
Neural Network) trained to classify the object found in an image.

A DNN is a construct of layers of neurons — similar to nodes in a graph, where
each layer learns certain patterns in the image. The first layers learn to
output simple patterns such as edges and corners, while the last layer outputs
the type of the object, e.g. dog, cat etc.

We chose to use the output of the layer before last — as that produced the best
results.

#### Database
The only component that was already available to us was Taboola’s internal
database of articles, containing, among other things, the thumbnail and the
article URL.

If such a database was not readily available to us, we could have just built our
own by [scraping](https://en.wikipedia.org/wiki/Web_scraping) images using a
library such as [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/).

#### Server
We used Flask as our web server. On startup, the server queries Taboola’s
internal database of articles from the web. It then sends each image to the
computer vision service, which returns an embedding. The embeddings are then
stored into a designated data structure called
[FAISS](https://code.facebook.com/posts/1373769912645926/faiss-a-library-for-efficient-similarity-search/)
(Facebook AI Similarity Search). It allows us to perform a [nearest neighbors
search](https://en.wikipedia.org/wiki/Nearest_neighbor_search).

Each image sent from the user is similarly transformed into an embedding. It is
then used as a query to the above data structure to retrieve its nearest
embeddings, meaning, images with similar patterns and content. Only images which
are considered similar above a predefined threshold are then returned to the
user.



So to recap, the entire architecture relies on three main components:

1.  web application
2.  computer vision service
3.  articles database



# Do it Yourself
![](images/hackathon_2018/diy.jpg)

The entire project was developed in under 36 hours by a team of 5 people.

This app touches several interesting and exciting domains that are “hot” in the
industry — AR and AI. It was a breeze to implement thanks to the commonly
available tools and libraries.

If there’s only one thing we want you to take from this is that it’s not that
difficult.

We invite you to be aware of the potential that lies in these domains and to be
on the lookout for interesting and exciting ideas. Once you find one, go and
have your own private hackathon!

---

# Finally
If you want to play around with the app, open
[https://zoom.taboola.com](https://zoom.taboola.com/) **on your phone** — use
Chrome on Android and Safari on iOS, and try hovering over different objects to
see the various results — keep in mind this is a pre-alpha hackathon project.

We want to thank our wonderful teammates who worked tirelessly to create this
amazing app –

[Amir Keren](https://www.linkedin.com/in/amir-keren/), [Yoel
Zeldes](https://www.linkedin.com/in/yoelzeldes/), [Elina
Revzin](https://www.linkedin.com/in/elinarevzin/), [Aviv
Rotman](https://www.linkedin.com/in/avivrotman/) and [Ofri
Mann](https://www.linkedin.com/in/ofri-mann-b53669/).

---

*Originally published at
[engineering.taboola.com](https://engineering.taboola.com/zooming-past-competition)
by me and [Amir Keren](https://medium.com/u/3214d1645ea8).*

