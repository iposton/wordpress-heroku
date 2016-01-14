### Description
This [website](http://blog-grolab.herokuapp.com/) is a wordpress(version 4.1.1) website hosted for free on heroku. It is based off of this [wordpress-heroku](https://github.com/mhoofman/wordpress-heroku) template that I cloned and used to deploy quickly. But first before deploy set up local development on [OS-X](https://github.com/mhoofman/wordpress-heroku/wiki/Setting-Up-a-Local-Environment-on-Mac-OS-X) or [Linux(Apache)](https://github.com/mhoofman/wordpress-heroku/wiki/Setting-Up-a-Local-Environment-on-Linux-(Apache)) and inside the themes directory add custom theme folder and install plugins related to the theme (important before you deploy to heroku because heroku will not unpack or write files so plugin installs need to be handled locally). I used The Ken Theme in this project. I recommend to not update wordpress as it may not be compatible with The Ken Theme. I was unsuccessful when I upgraded to the newest version of wordpress. 

### Software used in Grolab Blog
* MAMP (local development) [Download it launch free version](https://www.mamp.info/en/downloads/)    
* Postgres DB [Download it setup local DB ](http://postgresapp.com/) for OS-X
* The Ken Theme 
* [mhoofman/wordpress-heroku](https://github.com/mhoofman/wordpress-heroku) 
* AWS S3 (to store images see Setup below)
* Sendgrid (to send emails from heroku see Setup below)
* Wordpress
* Heroku

### Setup 
1. Clone [mhoofman/wordpress-heroku](https://github.com/mhoofman/wordpress-heroku)
2. Create/Log In to a Heroku account and Follow steps in mhoofman/wordpress-heroku readme(in short as follow)
    * <code>heroku create your-app-name</code>
    * <code>heroku addons:create heroku-postgresql</code> (free version)
    * <code>heroku pg:promote HEROKU_POSTGRESQL_INSTANCE</code> (the heroku-pg-instance is found after the word Creating... from step 2)
    * <code>heroku addons:create sendgrid:starter</code>
    * <code>git checkout -b production</code> (you will push to deploy from this branch but not yet)
    * Go [here](https://api.wordpress.org/secret-key/1.1/salt/) to get special wordpress keys and copy and paste the whole block to your <code>wp-config-sample.php</code> file over the empty block. Now stop deploy process and install theme plugins locally. 
3. [Setup DB to install wordpress locally.](https://github.com/mhoofman/wordpress-heroku/wiki/Setting-Up-a-Local-Environment-on-Mac-OS-X) 
4. Add Ken Theme folder to wp-content/themes
5. Sign In go to dashboard activate The Ken Theme install required plugins.(Envato toolkit and Visual Composer)
6. Back to mhoofman/wordpress-heroku readme steps
   * Configure Wordpress keys to heroku. example: 
  ``` 
  heroku config:set AUTH_KEY='put your unique phrase here' \
  SECURE_AUTH_KEY='put your unique phrase here' \
  LOGGED_IN_KEY='put your unique phrase here' \
  NONCE_KEY='put your unique phrase here' \
  AUTH_SALT='put your unique phrase here' \
  SECURE_AUTH_SALT='put your unique phrase here' \
  LOGGED_IN_SALT='put your unique phrase here' \
  NONCE_SALT='put your unique phrase here'
  ```
You can also go to your heroku account click on app/settings click on config vars and add them that way as well. 

    * Deploy the app! <code>git push heroku production:master</code>
7. Install wordpress sign in activate Ken and activate all the plugins. (There may be some funny text on the homepage after the visual composer plugin is activated which comes with The Ken Theme. See problems/solutions below.)

<h4>Setup AWS - S3 to store images on heroku</h4>
1. [Setup AWS account.](https://aws.amazon.com/)
2. Log In from AWS services dashboard select Identity & Access Management to create a user.
3. Click on Users then click on Create New User in input type nameofapp click create to generate special keys and save the keys keep in safe place on your computer for later. they will only appear once after create.(IMPORTANT DO NOT PUSH THESE KEYS TO A REPO IT IS AN EXPENSIVE MISTAKE)
4. Go to services dashboard click S3 and create a bucket. Name it and select US Standard region click create.
5. Make sure all permissions are checked and saved.
6. Back to dashboard go into IAM user account to attach special policy to user. Click on the user created. Click on policies then click on Create Policy then click on Copy an AWS Managed Policy then click on AdministratorAccess and edit it with this code:
```js
    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::BUCKETNAME",
                "arn:aws:s3:::BUCKETNAME/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "arn:aws:s3:::*"
        }
      ]
   } 
```
###### 7. Click Create Policy. Then attach it to the User Permissions.
###### 8. Store your AWS KEYS in your heroku account by going to the app/settings config vars click edit and name and insert keys. example use <code>AWS_ACCESS_KEY_ID</code> and <code>AWS_SECRET_ACCESS_KEY</code> this will be called from your <code>wp-config.php</code> file in your app so the variables should be named the same. This is the safe way to store keys don't put in your code.
###### 9. After thats configured navigate to AWS settings in your wordpress dashboard and it should list your bucket. Select and save. Now Images uploaded to your wordpress website should go to your bucket. Check your bucket to be sure photos are being stored there after you upload an image. If not something went wrong in config process. 

<h4>Setup Sendgrid to send emails from wordpress on heroku</h4>
1. When you addon sendgrid in deployment process it automatically saves your sengrid username password in app/settings config vars.
2. Simply copy those sendgrid vars and navigate to Sendgrid in wordpress dashboard and paste them in the username password inputs there and save.
3. Click test email and it will send an email to the address you installed wp with. If successful sendgrid config went well.

These steps above will get you a pretty nice wordpress site with about 10MB of free data to use. This process itself without much additions takes up about 7.5MB. Avoid uploading templates from Ken Theme as it will max out the data quickly. Just keep it simple or you can pay for more data. 


***

###Problems/Solutions
I had to deploy trial and error about 4 times before getting the best result. After trying to upgrade the latest wp things went bad. [mhoofman/wordpress-heroku](https://github.com/mhoofman/wordpress-heroku) suggests in his repo that upgrading wp is simple, but I did not find it so and I would not try upgrading. The Ken is not compatible with latest version of wp. 

I mentioned earlier above that you may get some funny text on homepage after activating visual composer plugin which comes with the Ken Theme. The fix for this issue is that heroku does not include the mbstring extension which is needed to run code located within the Ken Theme. I had to manually install this extension by creating a composer.json file in the root of the app directory. The Steps as follow...

```js
{
  "require": {
            "ext-mbstring": "*"
       }
}
``` 

1. I required the mbstring extension in the <code>composer.json</code> file: 
2. run <code>composer update</code> (this will create the composer lock file make sure you are on production branch)
3. <code>git push heroku production:master</code> 
Funny text should go away and homepage should load properly now. [This is my reference for the fix.](https://coderwall.com/p/deyqua/how-to-use-mbstring-on-heroku-php) 

###Refs
Here are some other wp-heroku repos that help users deploy wordpress to heroku.
https://ksylvest.com/posts/2014-05-02/deploying-wordpress-to-heroku
https://github.com/xyu/heroku-wp

I chose to use the mhoofman/wordpress-heroku setup because it seemed to be the quickest easiest setup that I could understand and it uses a local environment setup with MAMP. 

####[The Grolab Blog](http://blog-grolab.herokuapp.com/)