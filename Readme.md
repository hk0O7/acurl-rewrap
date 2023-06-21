# acurl-rewrap
This is a Bash function that acts as a wrapper for the official [acurl](https://docs.apigee.com/api-platform/system-administration/auth-tools#install) CLI utility for the Apigee Edge Cloud [Management API](https://apidocs.apigee.com/operations) (which is a `curl` wrapper itself).

Its main feature is to add the beginning of the full Management API endpoint URL to the relative paths you'd give it. This allows for a much more intuitive, practical and quicker usage while still being fully-compatible (and battle-tested with production changes) with the traditional (full) `acurl` URLs and parameters.

## Examples
Listing the virtualhosts in the `myorg` organization, `test` environment:
```
$ acurl /o/myorg/e/test/virtualhosts
```
To `GET`/`POST`/`PUT` payloads in XML (as often seen in the Edge UI) instead of JSON, you can use the `--XML` flag:
```
acurl /o/myorg/apis/sample-echo/revisions/1/proxies/default --XML
```

## Install
Add the following to your ~/.bashrc file (in your `$HOME` directory) on the host were you have [acurl and get_token](https://docs.apigee.com/api-platform/system-administration/auth-tools#install) installed:
```
acurl() {
	local acurl_args=("$@")
	local url_sub=1
	local i
	for ((i=0;i!=${#acurl_args[@]};i++)); do
		local -n acurl_arg=acurl_args[i]
		if grep -qE '^https?://[[:alnum:]]' <<< "$acurl_arg"; then
			url_sub=0
		elif [[ "$acurl_arg" == '--XML' ]]; then
			unset acurl_arg
			acurl_args+=('-H' 'Accept: application/xml' '-H' 'Content-Type: application/xml')
		elif grep -qE '^/((o(rganizations)?)|(mint))/.+' <<< "$acurl_arg"; then
			acurl_arg='https://api.enterprise.apigee.com/v1'"$acurl_arg"
		fi
	done
	"$(which acurl)" -sS --fail-with-body -w '\n' "${acurl_args[@]}"
}
```
Then reload the .bashrc (`source ~/.bashrc`) or just open up a new terminal and test it with a simple `GET` call (eg. `acurl /o/myorg`).
