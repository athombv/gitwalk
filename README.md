# gitwalk: Bulk processing of git repos

**gitwalk** is a tool to manipulate multiple git repositories at once. You select
a group of repos using an [expression](https://github.com/pazdera/gitwalk#expressions)
and provide an [operation](https://github.com/pazdera/gitwalk#processors) to be
completed. Gitwalk will download the repos, iterate through them and run the
operation for each one. This may be searching through files or the commit
history, editing and pushing the changes back upstream &mdash; whatever you can
think of.

![gitwalk in the terminal](https://raw.githubusercontent.com/pazdera/gitwalk/master/screenshot.png)

**gitwalk** is written in CoffeeScript and runs on Node.js. Most of the heavy
lifting in the background is done by [nodegit](http://www.nodegit.org/) and
[node-github](https://github.com/mikedeboer/node-github).

## Features

* Wildcard matching
* Integrates with GitHub API
* Works with private repos via GitHub auth
* Authenticated pushes via ssh and http
* Lets you define groups of repositories
* Built-in search tool
* Easily extensible with new [expressions](https://github.com/pazdera/gitwalk#expressions)
  and [processors](https://github.com/pazdera/gitwalk#processors)
* Usable from the CLI as well as JavaScript

## Installation

Gitwalk is distributed as an [npm](https://www.npmjs.com/package/gitwalk) package.
Use the following command to install it on your system:

```bash
$ npm install -g gitwalk
```

Make sure to include `-g` in case want to get the CLI command in your `$PATH`.

## Usage


```
gitwalk [options] <expr...> <proc> <cmd...>
```

```bash
$ gitwalk 'github:pazdera/tco' grep TODO
```

## Expressions

Expressions determine which repositories will be processed by the `gitwalk`
command. You can provide one or more of them and their results will be
combined. Check out the following examples:

```bash
# Matches all branches of the npm repo on GitHub.
$ gitwalk 'github:npm/npm:*' ...

# Matches all the git repositories in my home dir.
$ gitwalk '~/**/*' ...

# Use ^ to exclude previously matched repositories.
# Matches all my repos on GitHub _except_ of scriptster.
$ gitwalk 'github:pazdera/*' '^github:pazdera/scriptster' ...

# URLs work too.
$ gitwalk 'https://github.com/pazdera/tco.git:*' ...

# You can predefine custom groups of repositories.
# Check out the _Groups_ resolver below.
$ gitwalk 'group:all-js' 'group:all-ruby' ...
```

What comes after the last colon in each expression is interpreted as **branch
name**. If omitted, gitwalk assumes `master` by default. Also note that you can
use [globs](https://github.com/isaacs/node-glob#glob-primer) for certain parts.
Check out the detailed description of each resolver below for more information.

Under the hood, each type of expression is handled by a **resolver**. What
follows is a description of those that come with gitwalk by default. However,
it's really easy to add new ones. See the [contribution
guidelines](https://github.com/pazdera/gitwalk#contributing).

### GitHub

With this resolver, you can match repositories directly on GitHub. Gitwalk
will make queries to their API and clone the repositories automatically.

```
github:<username>/<repo>:<branch>
```

* **username**: GitHub username or organisation.
* **repo**: Which repositories to process (glob expressions allowed).
* _[optional]_ **branch**:  Name of the branch to process (glob expressions allowed).

#### Private repos

To be able to work with private repositories, you need to give gitwalk either
your credentials or auth token. Put this into your configuration file:

```json
{
  "resolvers": {
    "github": {
      "username": "example",
      "password": "1337_h4x0r",
      "token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    }
  }
}
```

#### Push access

Pushing to repositories on GitHub again requires authentication. You'll need
to give gitwalk either your SSH keys or credentials via the config file. Here's
how to do it:

```json
{
  "git": {
    "auth": {
      "github-http-auth": {
        "pattern": "^https?\\:\\/\\/github\\.com",
        "username": "example",
        "password": "1337_h4x0r"
      },
      "github-ssh-auth": {
        "pattern": "^git@github\\.com",
        "public_key": "/Users/radek/.ssh/id_rsa.pub",
        "private_key": "/Users/radek/.ssh/id_rsa",
        "passphrase": ""
      }
    }
  }
}
```

### Glob

You can match repositories on your file system using
[glob](https://en.wikipedia.org/wiki/Glob_(programming)). Note that gitwalk
will clone the repository even if it's stored locally on your computer. One
can never be too careful.

```
<path>:<branch>
```

* **path**: Location of the repositories (glob expressions allowed).
* _[optional]_ **branch**:  Name of the branch to process (glob expressions allowed).

### URL

URLs aren't a problem eiter. Gitwalk will make a local copy of the remote
repository and process it. However, you can't do any wildcard matching with
URLs.

```
<url>:<branch>
```

* **url**: An URL pointing to a git repository (glob **not** allowed).
* _[optional]_ **branch**:  Name of the branch to process (glob expressions allowed).

### Groups

You can define custom groups of expressions that you can refer to by name
later on. There are no predefined groups.

```
group:<name>
```

* **name**: Name of a group.

#### Configuring groups

Groups of expressions can be defined in the
[configuration file](https://github.com/pazdera/gitwalk#configuration) like
this:

```json
{
  "resolvers": {
    "groups": {
      "all-ruby": [
        "github:pazdera/tco",
        "github:pazdera/scriptster",
        "github:pazdera/catpix"
      ],
      "c": [
          "github:pazdera/*itree*:*",
          "^github:pazdera/e2fs*"
      ],
      "c++": [
        "github:pazdera/libcity:*",
        "github:pazdera/OgreCity:*",
        "github:pazdera/pop3client:*"
      ]
    }
  }
}
```

## Processors

### grep

### files

### commits

### command

## Configuration

Providing a config file isn't mandatory. However, you'll need one if you want
to be able to git-push or access private repositories. Gitwalk loads the
following files:

* `~/.gitwalk.(json|cson)`
* `/etc/gitwalk.(json|cson)`

Both `json` and `cson` formats are accepted. Here's how a basic gitwalk config
might look like:

```json
{
  "resolvers": {
    "github": {
      "token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    },
  },
  "git": {
    "auth": {
      "github-ssh": {
        "pattern": "^git@github\\.com",
        "public_key": "/Users/radek/.ssh/github.pub",
        "private_key": "/Users/radek/.ssh/github",
        "passphrase": ""
      }
    }
  },
  "logger": {
    "level": "debug"
  }
}%
```

## Javascript API

```js
var gitwalk = require('gitwalk');

gitwalk('github:pazdera/\*', gitwalk.proc.grep(/TODO/, '*.js'), function (err) {
    if (err) {
        console.log 'gitwalk failed (' + err + ')';
    }
});
```

## Contributing

## Credits

Radek Pazdera &lt;me@radek.io&gt;

## Licence

Please see [LICENCE](https://github.com/pazdera/gitwalk/blob/master/LICENCE).
