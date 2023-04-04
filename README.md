# nvim-http

You can use this plugin to run HTTP requests directly in your favourite text
editor.

This plugin is inspired by the
[vscode-restclient](https://github.com/Huachao/vscode-restclient) extension and
the [IntelliJ HTTP
Client](https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html).

It is a more advanced alternative to
[vim-http](https://github.com/nicwest/vim-http), but it comes closer to the
VSCode and IntelliJ alternatives because of the following features:

1. It supports for environment variables.
2. It uses Neovim's `async_call` mechanism, so the request is not blocking for
   the editor, and it can be stopped at any time.
3. An extended syntax for the `.http` files, with support for JSON and HTML
   blocks in the requests and the responses, as well as comments.
4. Real support for multi-request buffers - it matches the request that
   surrounds the cursor, while supporting visual selection as well.

## Installation

Add the plugin to your Neovim configuration through your favourite plugin
manager. For example:

```vim
Plug 'BlackLight/nvim-http'
```

### Dependencies

The plugin's core is in Python and it uses the
[`requests`](https://github.com/psf/requests) library to perform the requests.

The plugin therefore requires:

- The [`pynvim`](https://github.com/neovim/pynvim) bindings
  - `pip install pynvim`
  - On Arch-based distros: `[sudo] pacman -S python-pynvim`
  - On Debian-based distros: `[sudo] apt install python3-pynvim`
  - On Red Hat-based distros: `[sudo] yum install python-pynvim`
- The [`requests`](https://github.com/psf/requests) Python library
  - `pip install requests`
  - On Arch-based distros: `[sudo] pacman -S python-requests`
  - On Debian-based distros: `[sudo] apt install python3-requests`
  - On Red Hat-based distros: `[sudo] yum install python-requests`

## Usage

Open a buffer and add the body of your HTTP request. If the file name in the
buffer has the `.http` extension, then the plugin will automatically highlight
its syntax, and you can also set up nice automation on the file type.

To run your request:

1. Position the cursor anywhere in the body, or visually select the whole body.
2. Run the `:Http` command.

The response will be shown in a `vsplit` buffer.

If you want to interrupt the current request, use the `:HttpStop` command.

### Multiple requests

Like the `vim-http`, this plugin supports running requests both in normal and in
visual mode - in the latter case, only the selected range will be interpreted as
a request.

However, like the VSCode and IntelliJ plugins, it also supports multiple
requests in the same file in normal mode.

Any line starting with the triple hashtag will be interpreted as a request
separator:

```
### The first request

POST {{base_url}}/api/v1/users HTTP/1.1
Authorization: Bearer {{jwt_token}}

{
  "name": "Alice"
}

### The second request

GET {{base_url}}/api/v1/users?name=Alice HTTP/1.1
Authorization: Bearer {{jwt_token}}

### ...more requests
```

When `:Http` is run in normal mode, it will detect the boundaries of the
surrounding request and only execute that request.

### Environment files

This plugin is compatible with environment files generated by the VSCode
extension. Any file matching the `*.env.json` pattern stored in the same folder
as the `.http` file (or the current working directory) will be interpreted as an
environment file. Example format:

```json
{
  "dev": {
    "base_url": "http://localhost:3000",
    "jwt_token": null
  },
  "prod": {
    "base_url": "https://foobar.eu-west.some-cloud.com",
    "jwt_token": "SECRET"
  }
}
```

If multiple environment files are detected, or there are files with more than
one environment, then the `:Http` command will prompt the user for which
environment they want to use before executing the request.

The environment variables can then be wrapped as ``{{varname}}`` in the buffer:

```
GET {{base_url}}/api/v1/users?limit=10
Authorization: Bearer {{jwt_token}}
```

