## Task 1

> Explain how the load balancer behaves when you open and refresh the URL http://192.168.42.42 in your browser. Add screenshots to complement your explanations. We expect that you take a deeper a look at session management.

The first result when we call 192.168.42.42.

![](./screens/screen01.png)

After we refresh the page, we can see it's the other server that responds.

![](./screens/screen02.png)

Currently, there are no sessions and we can see after some refreshes that the sessionsViews are still 1 and don't increment. Cookies set by each servers are ignored by the load-balancer for the moment and because of that they cannot maintain a state between connections.

> Explain what should be the correct behavior of the load balancer for session management.

Normally, if sessions are successfully enabled, we should communicate with the same server every times after refreshing the page and the sessionsView counter should increment.

> Provide a sequence diagram to explain what is happening when one requests the URL for the first time and then refreshes the page. We want to see what is happening with the cookie. We want to see the sequence of messages exchanged (1) between the browser and HAProxy and (2) between HAProxy and the nodes S1 and S2. 

```sequence
participant Browser as b
participant HAProxy as h
participant S1 as s1
participant S2 as s2
b->h:GET /\nHOST:192.168.42.42 
h->s1:GET /\nHOST:192.168.42.11:3000
s1->h: {hello: "world!",...}
h->b: {hello: "world!",...}

b->h:GET /\nHOST:192.168.42.42 
h->s2:GET /\nHOST:192.168.42.22:3000
s2->h: {hello: "world!",...}
h->b: {hello: "world!",...}
```

> Provide a screenshot of the summary report from JMeter.

![](./screens/screen03.png)

> Clear the results in JMeter and re-run the test plan. Explain what is happening when only one node remains active. Provide another sequence diagram using the same model as the previous one.

As the server 1 is down, the load-balancer will redirect requests to the remaining alive servers, only S2 in our case.

![](./screens/screen04.png)

```sequence
participant Browser as b
participant HAProxy as h
participant S2 as s2
participant S1 as s1

b->h:GET /\nHOST:192.168.42.42 
h->s2:GET /\nHOST:192.168.42.22:3000
s2->h: {hello: "world!",...}
h->b: {hello: "world!",...}

b->h:GET /\nHOST:192.168.42.42 
h->s2:GET /\nHOST:192.168.42.22:3000
s2->h: {hello: "world!",...}
h->b: {hello: "world!",...}
```



## Task 2

```
backend nodes
	# Set SERVERID cookie for sticky-session
    cookie SERVERID insert indirect nocache

    # Define the list of nodes to be in the balancing mechanism
    # http://cbonte.github.io/haproxy-dconv/2.2/configuration.html#4-server
    server s1 ${WEBAPP_1_IP}:3000 check cookie s1
    server s2 ${WEBAPP_2_IP}:3000 check cookie s2
```

