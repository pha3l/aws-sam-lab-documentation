# Deployment
If all has gone according to plan, you should now have a working application communicating with the serverless API.

Stop the angular dev server. Run the following to generate the deployment package:
```bash
$ ng build --prod
```

Let's create a bucket for the site:
```bash
$ aws s3api create-bucket --bucket <YOUR_BUCKET_NAME> --region us-west-2 \
   --create-bucket-configuration LocationConstraint=us-west-2
```
*remember to choose a unique name for the bucket*

Enable s3 static site hosting:
```bash
$ aws s3 website s3://<YOUR_BUCKET_NAME> --index-document index.html
```

Sync the production files from the `/dist` directory to the bucket, also setting public-read access control lists for each file:
```bash
$ aws s3 sync ./dist s3://<YOUR_BUCKET_NAME> --acl bucket-owner-full-control --acl public-read
```

That's it! You should be able to visit your application at `http://<YOUR_BUCKET_NAME>.s3-website-us-west-2.amazonaws.com`

---

We've made it to the end.  There isn't any more.  Thanks for participating and congrats if you made it this far!  I hope that you are as excited about this tech as I am! It's 3AM and I'm exhausted! Ciao!