---
title: Getting started with Jekyll
date: 2025-02-19 09:11:43
tags: [jekyll, website]
---

## What is Jekyll

Jekyll is a (relatively) simple to use tool for generating static websites. It's perfect for blogs as it takes care of the boring side of creating a website and leaves you with just the fun of writing the posts.

What makes it perfect is the security gains from not having a dynamic website along with databases, admin interfaces, etc. that can be and usually will be exploited by the hostiles. What's more, the blog itself can be easily hosted for free on [Github Pages](https://pages.github.com/).

[Jekyll website](https://jekyllrb.com/) \
[Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy)

## Preparation

I'll be using an LXC container on Proxmox as the development environment to have a Linux based system and to keep my own computer cleaner. Besides having a separate environment for this purpose alone, allows me to easily start from scratch, should anything go wrong or not work as intended, further down the line.

### LXC container

> Adding the SSH key at the setup stage makes things easier later.
{: .prompt-tip }

- **Template**: debian-12-standard_12.7-1_amd64.tar.zst
- **Storage**: 8 GiB
- **CPU Cores**: 1
- **Memory**: 1024 MiB

### Development environment

[Official instructions for Ubuntu](https://jekyllrb.com/docs/installation/ubuntu/)

Update & upgrade the packages in the container.

```shell
apt update && apt upgrade -y
```

Install prerequisites.

```shell
apt install ruby-full build-essential zlib1g-dev git
```

Official instructions suggest to avoid installing RubyGems packages as a root user, however as this is a separate environment for Jekyll alone it doesn't matter.

```shell
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Install Jekyll and Bundler.

```shell
gem install jekyll bundler
```

Generate an SSH key.

```shell
ssh-keygen -t ed25519
```

Configure Git.

```shell
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

### Github repository

1. Go on the theme's [Starter repository](https://github.com/cotes2020/chirpy-starter) and click on `Use this template` followed by `Create a new repository` on top right.
   
1. Add repository name. To host the website on Github Pages, the repository name has to be in the format `<github-username>.github.io`.
   
1. To be able to host the page on Github Pages, the repository would have to be `Public`.
   
1. Navigate to repository settings and add a new key with write access, that was previously generated, in the `Deploy keys` section.
   
   > The key can also be added under the profile settings, but it is recommended to add it to repository to minimise the scope of the key.
   {: .prompt-info }
   
    ```shell
    cat ~/.ssh/id_ed25519.pub 
    ```

### VS Code

Connect to remote host with VS Code and install the recommended addons, which make life a little easier, on the remote host and change some settings.

#### Recommended addons

- Print Timestamp
- Markdown All in One

#### Addon settings

- `Markdown › Copy Files: Destination`

    This option let's you choose where pasted in images are saved. For instance in the `/assets/img/posts/<document-name>/` folder to make managing files easier.
    
    > Make sure to add it to remote host so it would not interfere with settings on other machines.
    {: .prompt-tip }

    Add item:

    - **Item**: `*`
    - **Value**: `../assets/img/posts/${documentBaseName}/${fileName}`

- `Markdown › Editor › Paste Url As Formatted Link`

    - Change `smartWithSelection` to `smart`

- `Printtimestamp: Timeformat`
  
    - Change time format to `YYYY-MM-DD HH:mm:ss` because the default **YYYY-DD-MM** does just not make any sense. Besides having month after year helps sorting files in chronological order.

## Site setup

Clone the new repo to the development environment using SSH.

```shell
git clone git@github.com:<github-username>/<github-username>.github.io.git
```

Navigate to the repo folder and install theme dependencies.

```shell
cd <github-username>.github.io
bundle
```

### Site configuration

- Modify `_config.yml` to fit the need. Pay attention to following settings:

  - `timezone`
  - `title`
  - `tagline`
  - `description`
  - `url`
  - `social`
    - `name`
    - `email`
    - `links`
      - Defines what will be the copyright link. Uncomment all and add `/about` to link it to About page
  - `analytics`
  - `avatar` - either link from internet or place it in `assets/` folder and use relative path as a link.
  - `baseurl` - leave it as `""` if the site is hosted at **example.com** rather than **example.com/blog**
  
- Modify `data/contact.yml` to select which social media and contact options would be available on the site.
- File `_data/share.yml` has sharing options for the articles.
- Modify `_tabs/about.md` to include content for About page.

#### Favicon

1. Use [favicon generator](https://realfavicongenerator.net/) to create the favicon.
1. Download the favicon package and extract it.
1. Copy **only** `*.png`, `*.ico` and `*.svg` files to `assets/img/favicons/` (if the folder does not exist, create it).

#### Modify footer

To modify the footer, download `_includes/footer.html` from [Chirpy repository](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/_includes/footer.html) and place it in the same location.

#### Add or remove navigation items

Navigation items or extra pages are located in `_tabs/`.

- To remove one, just delete the file.
- To add, create a new markdown file in the directory and add the [front matter](https://jekyllrb.com/docs/front-matter/) to the file along with the [icon](https://fontawesome.com/v4/icons/) and the order it should appear on the left hand side:

  > Default layout is `page`
  {: .prompt-info }
  
  ```md
  ---
  icon: fas fa-cart-plus
  order: 5
  ---
  ```

## Deploy initial site

1. Navigate to `Repository settings` and select `Pages` on the left side menu.
1. From the `Source` selection, select `Github Actions`.
1. Push the changes to Github.

The deployment workflow can be observed and troubleshooted from `Actions` menu in Github. \
Now, if all goes according to plan, the website should be accessible on `https://<github-username>.github.io` or on the custom domain set up for the website. How to set up a custom domain for your page, can be found in [Github Docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site).

## Creating a post

[All possible page elements](https://chirpy.cotes.page/posts/write-a-new-post/) \
[Text and typography](https://chirpy.cotes.page/posts/text-and-typography/)

1. Create a new markdown file in `_posts` directory with the format `YYYY-MM-DD-title.md`
1. Add the front matter to the file.

    >Changing the date and time in the front matter is where the [Print Timestamp](#recommended-addons) addon comes in handy. Using the [command palette](https://code.visualstudio.com/api/ux-guidelines/command-palette), select `Print Timestamp`. The timestamp is in UTC and will be shown depending on the timezone chosen in the `_config.yml` file.
    {: .prompt-tip }

    ```md
    ---
    title: TITLE
    date: YYYY-MM-DD HH:MM:SS
    categories: [TOP_CATEGORY, SUB_CATEGORY] # Remove if not using categories.
    tags: [tag, tag2, tag3] # TAG names should always be lowercase
    ---
    ```

1. Write the content.

## Preview site

Run the following command in the project directory to have a live preview of the site. VSCode takes care of the port forwarding so even if the development environment is in the LXC container, site can still be accessed on [http://127.0.0.1:4000](http://127.0.0.1:4000).

```shell
bundle exec jekyll s
```

## Publish

Once happy with the post, just commit and push the changes to Github.

**_Voilà!_**