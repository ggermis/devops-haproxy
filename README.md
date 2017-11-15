# Description

Playground for testing HAProxy behaviour. 
Uses ansible to configure the HAProxy server.
This will configure 4 virtualbox instances:

* haproxy-01 (33.33.33.11) -> HAProxy server
* backend-01 (33.33.33.21) -> Server from backend pool which can run a service
* backend-02 (33.33.33.22) -> Server from backend pool which can run a service
* backend-03 (33.33.33.23) -> Server from backend pool which can run a service

Services preconfigured on HAProxy are on ports 1001 and 1002 for all backend servers

* service-01 -> port 1001
* service-02 -> port 1002

HAProxy (33.33.33.11) will forward traffic received on the port the service is running on to an active backend



# How to test ?

```bash
$ vagrant up
$ vagrant ssh backend-01
$ vagrant ssh backend-02
$ vagrant ssh backend-03
```

Simulate a service with the following commands

```bash
$ export PORT=1001
$ while true ; do  echo -e "HTTP/1.1 200 OK\n\n $(date)" | nc -l -p "$PORT"  ; done
```

Look at the HAProxy stats page: http://33.33.33.11:1111/haproxy?stats

```bash
loop_curl () {
	local URL="$1"
	local SLEEP="${2:-1}"
	while true
	do
		if [[ -n $DEBUG ]]
		then
			HTTP_CODE="$(curl -s -o/dev/null -w "%{http_code}" "$URL")"
			(( HTTP_CODE != 200 )) && echo -n "[${HTTP_CODE}]" || echo -n "."
		else
			curl -fs "$1" -o/dev/null && echo -n "." || echo -n "F"
		fi
		sleep "$SLEEP"
	done
}
```

To see how HAProxy handles the requests

```bash
$ loop_curl http://33.33.33.11:1001 .1
```