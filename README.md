# Light Casts Shadows Static Site

> This is the main site that hosts Light Casts Shadows content when/as the project progresses

This is the source code for this particular site but you can use it as a starting point for your own site easily. Simply remove the text content and update stylesheets as needed.

> This is the source for Phase 1: Production Log & Hype Building

## Development setup

```
nvm use # install nvm if needed or, if you know what you're doing, use the version of Node set in the .nvmrc file
npm install
```

### Dev server

*__HINT:__* Look at the `scripts` object in `package.json`. There are some useful tasks that can be automated away for you like creating a new post, generating the blog locally, and running the dev server so far.

Run `npm run start:dev`. A full site will spin up. Be warned, this site is set up assuming that the blog section is actually hosted outside of the repository and this is why the "Production Log" link will always 404. You'll need to manually go to `/blog/` to see the blog. To be able to easily prune the repo without destroying the production blog, the `generate_prodlog.js` will move Matkdown post generated into HTML into a separate directory on the server which Nginx is set up to recognize as a path and server files from. In this case that path on the server is `/path/to/website directories/__production_log__/`. If you completely duplicate this site but just change the content then this is where it will be for you as well but you'll need to update your Nginx `sites-available` conf with a location block  that points to your blog, update the references to `production_log` throughout the project (do a find and replace on the source folder for that), and you'll need to update the `LCS_LOG_DIR` environment variable in the git post-receive hook script to point to your particular blog directory.  This is only relevant for deploying to production. In development you'll need to manually navigate to any blog posts (for now) because the generation script uses templates that point to the production folder. *This is on the list of items to enhance*.''

### How the Blog ("Production Log") Works

The production log works strangely because the static nav points to the production server but the dev server does not. Here's how it works (assuming you've completed the Deployment setup steps).

We don't really want or need to host all the HTML files on our local server, especially when the blog gets large. So whenever the master branch is deployed to your production remote the `post-receive` hook will run and deploy the static site as usual. The last step of the process is running `source/workers/generate_prodlog.js`. That script will loop through all `.md` files in `source/blog/drafts/`, parse the Markdown into HTML, write that HTML to a file in whatever directory your set as your `LCS`

## Deployment

> Read all the instructions once all the way through before beginning the steps. This will make it much easier to understand

A post-receive hook should be added to your remote server that checks out code from the master branch and copies the Webpack built files to the site’s public directory. This hook also generates the blog.

You’ll need to install `nvm` so you can install the version of Node you’re developing with (or install Node any other way if you're experienced). After that run `nvm install XX.XX.X` where the `X`’s are the version of node you need to install.

1. Log into your server and create a blank repository. Your normal user account should suffice and it’s suggested that you have a folder dedicated to bare repos there, Run this which is a series of commands that will create a new folder to hold your personal, privately hosted git repos and initialize what you need: `mkdir -p git/lcs_static.git && cd $_ git init --bare` (note there are two dashes before `bare`).
2. Enter the hooks directory (`cd ~/git/lcs_static.git/hooks/`) for create the deploy hook, My production branch is called `master` but yours may be different depending on how you configured it in GitHub and how the new initial branches are called `main` in GitHub, I like `master` so I changed it. Once in the `hooks` directory, `nano post-receive` and copy this script into the empty file. You’ll need to change parts of it to fit your particular deployment but that should be obvious and will be pointed out:

*Below:* Contents of `post-receive`
```
#!/bin/bash
while read oldrev newrev ref
do
    if [[ $ref =~ .*/NAME_OF_BRANCH_YOU_WANT_TO_DEPLOY$ ]];
    then
        echo "NAME_OF_BRANCH_YOU_WANT_TO_DEPLOY ref received.  Deploying NAME_OF_BRANCH_YOU_WANT_TO_DEPLOY branch to production..."
        git --work-tree=/home/YOUR_USER/lcs_static_raw/ --git-dir=/home/YOUR_USER/git/lcs_static.git/ checkout -f
        cd /home/YOUR_USER/lcs_static_raw/ && export PATH=$PATH:/home/YOUR_USER/.nvm/versions/node/v16.14.2/bin/ && npm install && npx webpack --mode production
        rm -R /srv/www/lightcastsshadows/public/* && cp -R ~/lcs_static_raw/build/* /srv/www/lightcastsshadows/public/ # NOTE: YOU'LL NEED TO CREATE THE NGINX  CONFIG AND  FOLDER STRUCTURE  IT REFERENCES
        echo "Static site deployed. Checking for new production log posts..."
        # Generate the blog using the node script in production
        NODE_ENV=production LCS_LOG_DIR=/srv/www/lightcastsshadows/production_log/ /home/YOUR_USER/.nvm/versions/node/v16.14.2/bin/node /srv/www/lightcastsshadows/public/workers/generate_prodlog.js # NOTE: YOUR PRODUCTION_LOG FOLDER CAN BE ANYTHING YOU WANT. JUST CHANGE THE NAV REFERENCE IN THE HTML AND YOUR NGINX CONFIG
        echo "Production log generation script run. If errors occured you will see them above this message."
    else
        echo "Ref $ref successfully received.  Doing nothing: only the master branch may be deployed on this server."
    fi
done
```

*Below:* Location to be added to Nginx `server { ... }` block. You'll add this below the `location / { ... }` block. Putting this block too early will result in you being unable to get to the homepage

```
# You don't need to call this subdirectory 'production_log'. If you change it just be sure to change all references to it in the site source, post-receive hook, and nginx site config
location /production_log {
  alias /srv/www/lightcastsshadows/production_log/;
  index index.html;
  try_files $uri $uri/ /log/index.html =404;
}
```

3. Create a folder to hold temporary copy of your site before it gets copied to the Nginx public folder in your home folder. Mine is simply `lcs_static_raw`
4. Follow these steps *exactly to build and deploy your site each time your push to your remote production branch*. Your remote `production` branch will not be on GitHub. You will need to use the blank repository we made in an earlier step on your own server as your production repository. Note that this will only deploy when you push your main/master branch to your production remote.
  - In your local termonal (on your computer, within the repository) run `git remote add production YOUR_USER_NAME@XYOUR_IP_ADDR:~/git/lcs_static.git`
  - Now you can  push to your remote git repo that is not on github by running `git push production BRANCH_NAME`
    - Note here that I keep an exact copy of all my branches on both GitHub and my personal server. The only difference is that the post-receive hook only turns on when you are pushing your main or master branch to your production remote
  - Update your `post-receive` hook to look like the one above and be sure to customize it...

### Reporting issues

Please create issues and pull requests any time. The project has a Kanban board and I'm running it using Agile and scrume (I am a senior software engineer, Certified Scrum Master, and tech lead/project manager at my day job -- I don't do all those things at once but I've been in those roles professionally over the course of my working career) so I'll be handling it that way.

### FAQ/Info

This was written at the very beginning of the project. It will be updated. Do feel free to open issues for questions.
