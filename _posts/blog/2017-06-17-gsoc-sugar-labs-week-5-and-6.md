---
title: GSOC Sugar Labs - Week 5 and 6
categories: blog
excerpt: 'GSOC Suar Labs - Week 5 and 6 '
tags: [programming,sugar labs,gsoc]
description: 'GSOC Suar Labs - Week 5 and 6 '

---
This is the fifth and sixth post (combined into one) in the series of my weekly GSOC Sugar Labs, where I summarize my week of working with [Sugar Labs](https://www.sugarlabs.org) under GSOC.
This post was published little late :sweat_smile: due to the delays on my side, since I wanted to merge merge this and next weekly post.

## Weekly Journal 

In the following bi-weekly period I worked on the aslo-v3 along with Samuel. Samuel and I disucssed potential architecture and work flow. Intial plan was to complete database less but later we relaized, a small database layer is needed to index packages. We settled on MongoDB because of it's No-SQL capabilities and relaxing Schemaless architecture seems perfect for storing the activity meta data and translations. We re-iterated over workflow over and over and intial plan was to start with something simple. We started with a simple and high level which involved 

<blockquote>
1. Parse activity.info (forget about gtk3, etc).

2. Store in DB: activity name and version (you might look activity table in current ASLO).

3. Build bundle.

4. Show entry in web page.
</blockquote>

Bundle_id would be used to uniquely identify identify which is just a Fully Qualified Package Name, similar to what Java does.
A quick solution to convert activity meta data into db documents was to parse activity.info into a json and push the raw json directly into Mongo. 
A [typical activity.info](https://github.com/sugarlabs/activity-turtleart-gtk3/blob/master/activity/activity.info) looks something like this. 
This one is of [Turtle Art activity](https://github.com/sugarlabs/activity-turtleart-gtk3/blob/master/activity/activity.info)

{% highlight python %}
[Activity]
name = TurtleBlocks
activity_version = 213
license = MIT
bundle_id = org.laptop.TurtleArtActivity
exec = sugar-activity TurtleArtActivity.TurtleArtActivity
icon = activity-turtleart
show_launcher = yes
website = http://wiki.sugarlabs.org/go/Activities/Turtle_Art
mime_types = application/x-turtle-art;application/vnd.turtleblocks
categories = programming art
summary = A Logo-inspired turtle that draws colorful pictures with snap-together visual programming blocks
{% endhighlight %}


Which is a simple config file and can easily parsed by [ConfigParser](https://docs.python.org/3/library/configparser.html)  and conversion to JSON was simple.
MetaData to Json conversion can be visualized as 

{% highlight bash %}
Parse MetaData ---> Convert to Python Dict ---> Use InBuilt JSON Parser to convert Dict into JSON
{% endhighlight %}

{% highlight python %}
## Very primitive code for parsing and converting metadata
def read_activity(activity_file):
    parser = ConfigParser.ConfigParser()
    parser.read(activity_file)
    return parser

def convert_to_json_string(parser):
    # Get all attrbutes of acitvity
    attributes = parser.items('Activity')
    # Convert attributes to a JSON string
    return json.dumps(dict(attributes))

def convert_to_json_object(parser):
    return json.loads(convert_to_json_string(parser))
{% endhighlight %}

To interact with MongoDB, we intially went with Flask-Mongo but then changed it to pymongo upon which Flask-Mongo is built for more fine control. 


Next was to build the actual bundles which is what end users expect to download. Now building part was little tricky because the [Sugar Toolkit](https://github.com/sugarlabs/sugar-toolkit-gtk3) has no documentation. So I looked at SamP's code to find how toolkit was installed. I expressed my concerns regarding with the community and a new pip package might become a reality in the coming time.  There are two versions of Sugar Toolkit - old version which supported GTK2 only and a new GTK3 version. New Toolkit (GTK3) is all we need to build packages. 
Since there is no pip package for toolkit yet. To install we need to move the source to `dist-packages` folder and create a symlink of `sugar` to `sugar3` to build old gtk2 packages. 

[SamToday's DockerFile](https://github.com/samdroid-apps/aslo/blob/master/activity-build-docker/Dockerfile#L31/) had the instructions so it was pretty easy to follow along. Sadly toolkit is built onyl for Python2, so it won't work with Python3.
{% highlight bash %}
# Do a shallow clone
git clone --depth=1 https://github.com/sugarlabs/sugar-toolkit-gtk3; \
# Move
mv sugar-toolkit-gtk3/src/sugar3 /usr/lib/python2.7/site-packages/sugar3; \
# Clean up the cloned floder
rm -rf sugar-toolkit-gtk3/
# SymLink Sugar and Sugar3
ln -s /usr/lib/python2.7/site-packages/sugar3 /usr/lib/python2.7/site-packages/sugar
{% endhighlight %}

Once we have the toolkit installed, building bundles is easy as 
`python setup.py dist_xo --no-fail`

Aslo-v3 was written in Python2 when I started but Samuel suggested (a right suggestion) to move to Python3, since it was early stage it was quite easy to move from Python2 to Python3, only change was to use new ConfigParser and a slighltly different instantion instructions. One advantage of `virtualenv` is how easy it is to switch between interpreters along with dependencies without touching files outside /home.
 
Next move was to dockerize the build for security reasons. I looked at SamToday's code and migrated from `fedora` build image to `python-minimal` image and since it's based upon Debian, we can still install foriegn packages (so we get best of both worlds). 

{% highlight DockerFile %}
# Use Slim variant of Python from Docker Hub
FROM python:2.7-slim

# Update Repos and Install git
RUN  apt-get update\
     && apt-get install -y git gettext

# Thanks to  https://github.com/samdroid-apps/aslo/blob/master/activity-build-docker/Dockerfile#L24 for the instructions
# Clone the Sugar Toolkit
RUN git clone https://github.com/sugarlabs/sugar-toolkit-gtk3 --depth=1; \
    mkdir -p /usr/local/lib/python2.7/site-packages;  \
    mv sugar-toolkit-gtk3/src/sugar3 /usr/local/lib/python2.7/site-packages/sugar3; \
    rm -rf sugar-toolkit-gtk3

RUN rm -rf /usr/local/lib/python2.7/site-packages/sugar; \
    ln -s /usr/local/lib/python2.7/site-packages/sugar3 /usr/local/lib/python2.7/site-packages/sugar


# Copy build script to container
COPY build.sh $PWD


# Thanks to https://forums.docker.com/t/is-it-possible-to-pass-arguments-in-dockerfile/14488/3 for the entrypoint tip
ENTRYPOINT ["/build.sh"]
{% endhighlight %}

My approach was to use spawn a docker container build from a the above Dokcerfile, building docker image is easy as doing `docker build . -t ` provided Dockerfile is also in the same directory. We pass name of the activity along with docker spawn command and open two volume directories to interact with the host , to build the activity and move the activity to other folder for serving, script provided in the Entypoint does the actual building.

{% highlight bash%}
#!/bin/bash
# Builds a sugar activity, accpets repo_name as parameter
set -e
# Temporary bash activation
#source "$HOME/venvs/sugar-dev/bin/activate"
target_repo=$1
#echo $VIRTUAL_ENV
echo "Target repo is $target_repo"
cd "/activities/$target_repo"

echo "Creating Bundle"
python setup.py dist_xo
#mkdir -p bundles
#mv dist/*.xo "../../bundles/"
echo "Moving Bundle"
mv dist/*.xo /bundles/

echo "Cleaning up"

rm -rf "activities/$target_repo"
{% endhighlight %}

What gave me hard time was the permission managment between docker and host. Since Folders where activities and bundles are stored are owned by the aslo user which runs the backend server but Docker runs as root and the final image copied to folders have different (root) permissions and cannot be accessed/modified by the actual user, this was a big issue and I managed to solve it by using `setfacl ` and running docker as current user by passing current user id to docker which then emulates the user actions under the user id. 
Also any file of a shared volume deleted under docker will still remain in the host system but you can change permissions and can move files :angry:
{% highlight DockerFile %}
# Use Slim variant of Python from Docker Hub
FROM python:2.7-slim

# Update Repos and Install git
RUN  apt-get update\
     && apt-get install -y git gettext

# Thanks to  https://github.com/samdroid-apps/aslo/blob/master/activity-build-docker/Dockerfile#L24 for the instructions
# Clone the Sugar Toolkit
RUN git clone https://github.com/sugarlabs/sugar-toolkit-gtk3 --depth=1; \
    mkdir -p /usr/local/lib/python2.7/site-packages;  \
    mv sugar-toolkit-gtk3/src/sugar3 /usr/local/lib/python2.7/site-packages/sugar3; \
    rm -rf sugar-toolkit-gtk3

RUN rm -rf /usr/local/lib/python2.7/site-packages/sugar; \
    ln -s /usr/local/lib/python2.7/site-packages/sugar3 /usr/local/lib/python2.7/site-packages/sugar

#RUN useradd --create-home -s /bin/bash simpleuser

#USER simpleuser
# Copy build script to container
COPY build.sh $PWD


# Thanks to https://forums.docker.com/t/is-it-possible-to-pass-arguments-in-dockerfile/14488/3 for the entrypoint tip
ENTRYPOINT ["/build.sh"]
{% endhighlight %}

{% highlight bash %}
#!/bin/bash
# Builds a sugar activity, accpets repo_name as parameter
set -e
# Temporary bash activation
#source "$HOME/venvs/sugar-dev/bin/activate"
target_repo=$1
#echo $VIRTUAL_ENV
echo "Target repo is $target_repo"
cd "/activities/$target_repo"
echo "Bridge USER ID is $LOCAL_USER_ID"
echo $USER 
echo "Creating Bundle"
python setup.py dist_xo
mkdir -p bundles
#mv dist/*.xo "../../bundles/"
echo "Moving Bundle"
# Assign target repo 
setfacl -R -m u:$LOCAL_USER_ID:rwx "/activities/$target_repo"
cp dist/*.xo /bundles/

echo "Setting up"
#setfacl -R -m u:$LOCAL_USER_ID:rwx $target_repo
#rm -rf "activities/$target_repo"
{% endhighlight %}

Docker part was the biggest time consuming part :weary:

In between I did a rough sketch of UI 
<img src="/images/gsoc-week-5-and-6/mockup.png" alt="ASLO Mockups with Cats">

Samuel also suggestes to clone only the latest tag branch instead of master. Also bundles can be as big as 150 MB we looked on the file size limits for Github releases and [as per documentation](https://help.github.com/articles/distributing-large-binaries/) single file can be as big as 2 GB :grin: . We also discussed using Github status api in the coming future to inform users of build status and a way to support screenshots , i18n/l10n.

## Goals for Next Week
I plan on to fix and improve ASLO to make it ready for the first GSOC review.
If you find any mistakes/typos, do let me know and I'll fix them.



