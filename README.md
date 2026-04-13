# ://fmount

This blog is built starting from the responsive jekyll theme Harmony. 

#### How to edit this website

1. **EDIT LOCAL, VIEW ONLINE**
    * Download the [GitHub Desktop Client](https://desktop.github.com/) and sync the repo
    * Edit the files and push to GitHub when finished
  
2. **EDIT AND PREVIEW LOCAL**
    * Clone the repo
    * Run Jekyll locally via Podman (see below)
    * Edit the files
    * Preview edits at localhost:4000
    * When finished, push to GitHub

#### Running locally with Podman

Add this helper function to your shell (e.g. `.zshrc` or `.bashrc`):

```bash
jekyll() {
    if (( $# == 0 )); then
        echo "usage: jekyll [blog-path] ..."
        return 1
    fi
    podman run --rm -it \
      -v "$1:/srv/jekyll:Z" \
      -e JEKYLL_ROOTLESS=1 \
      -e BUNDLE_PATH='/srv/.bundle' \
      -p 4000:4000 \
      jekyll/jekyll bash -c "bundle update && jekyll serve --watch --drafts --future"
}
```

Then run it passing the path to the blog repo:

```bash
jekyll /home/user/fmount.github.io
```

The site will be available at [http://localhost:4000](http://localhost:4000). Changes to files are picked up automatically thanks to `--watch`.

&nbsp;

#### Creating a new post

1. Open the posts folder

2. Create a new file named **YYYY-MM-DD-title-of-the-post.md**

3. Insert the front matter:
   * **layout:** define the page layout. Default value is "post".
   * **title:** the title of the post. Make it SEO friendly and put it into quotation marks: "Example Post"
   * **date**: the creation date. The format should be: YYYY-MM-DD HH:MM:SS
   * **last_modified_at**: last edited date. The format should be: YYYY-MM-DD HH:MM:SS
   * **excerpt**: the summary of the post. Make it SEO friendly and put it into quotation marks: "Description"
   * **categories**: category-name (it could be: computing, support, logs, object-storage, networks, cdn, dns-domains, projects, tags, add-ons, reports, admin-portal)
   * **tags**: tag-name (optional)
   
4. Edit the post content with Markdown. For a Syntax reference, see <http://kramdown.gettalong.org/quickref.html>

5. If the post has images, put them into the folder assets/images/posts

6. For additional examples, custom styles and more check the **example-post.md** file in the **_drafts** folder.

