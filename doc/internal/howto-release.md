How to cut a Newsboat release
-----------------------------

This document describes all the steps one should go through in order to make
a new release. The person doing the release should have push access to the
master repository and shell access to newsboat.org.

1. Update CHANGELOG
    * Consult `git log --reverse PREVIOUS_VERISION..`
    * Mention issue number ("#X" for Newsboat issues, full link to issue tracker
        for Newsbeuter issues)
    * Mention the name of contributor
    * Acknowledge contributions from people whose changes didn't make it into
        the lists
2. Update version in config.h
3. Create new tag:
    * `git tag --sign -u 'newsboat@googlegroups.com' rVERSION`
    * First line: "Release Newsboat VERSION"
    * Description: copy of changelog entry
        * Don't use "###" style for headers because they'll be stripped ("#" is
            a shell comment). Use "===" style instead
4. Prepare the repo for the next release
    * Add "Unreleased" section to CHANGELOG
5. Prepare the tarball
    * Clone your local clone somewhere else
    * In that new clone, `rm -rf .git`
    * Rename that clone to newsboat-VERSION
    * Pack that clone in a tarball:
        `tar cvJf newsboat-VERSION.tar.xz newsboat-VERSION`
    * Sign the tarball:
        `gpg2 --sign-with 'newsboat@googlegroups.com' --detach-sign --armour newsboat-VERSION.tar.xz`
    * Upload both files to newsboat.org staging area
6. Prepare the docs
    * In your local clone: `make -j5 doc`
    * Upload contents of `doc/xhtml/` to newsboat.org staging area
7. Publish the release
    * Prepare the directory on the server
        * `cp -rfv /usr/local/www/newsboat.org/www/ newsboat`
        * Prepare directories: `mkdir -p newsboat/releases/VERSION/docs`
        * Move tarball and its signature:
            `mv newsboat-VERSION* newsboat/releases/VERSION/`
        * Prepare docs:
            `gzip --keep --best docbook-xsl.css faq.html newsboat.html`
        * Move docs:
            `mv docbook-xsl.css* faq.html* newsboat.html* newsboat/releases/2.10.2/docs/`
        * Edit `index.html`
            * Move current release to the list of previous releases
            * Update current release version
            * Update current release date
            * Update current release links
            * Gzip the result: `gzip --best --keep --force newsboat/index.html`
        * Edit `news.atom`
            * Update `<updated>` field of the channel
            * Use the same date-time for `<published>` and `<updated>` in new
                `<entry>`
            * Update entry's `<link>` and `<id>` to point to new docs'
                `newsboat.html`
            * `<title>`: "Newsboat VERSION is out"
            * Gzip the result: `gzip --best --keep --force newsboat/news.atom`
    * Prepare an email to the mailing list
        * Same topic and contents as in `news.atom` entry
    * Deploy the directory on the server:
        `sudo cp -rv newsboat/ /usr/local/www/newsboat.org/www/ && sudo chmod -R a+r /usr/local/www/newsboat.org/www/`
    * Push the code: `git push && git push --tags`
8. Tell the world about it
    * Send an email to the mailing list
    * Change the topic on #newsboat at Freenode
