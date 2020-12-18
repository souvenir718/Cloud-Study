# kube-proxy

  쿠버네티스에서 서비스를 만들었을 때 나오는 ClusterIP나 NodePort로 접근하게 해주는건 **kube-proxy**다. **kube-proxy**는 쿠버네티스 클러스터의 각 노드마다 실행되고 있으면서 클러스터 내부 IP로 연결되기 바라는 요청을 적절한 곳으로 전달해주는 역할을 한다. **kube-proxy**가 네트워크를 관리하는 방법은 **userspace**, **iptables**, **ipvs** 3가지가 있다. userspace - iptables - ipvs 모드순으로 넘어가고 있다.



## userspace 모드

![](data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4KPCEtLSBHZW5lcmF0b3I6IEFkb2JlIElsbHVzdHJhdG9yIDE5LjIuMSwgU1ZHIEV4cG9ydCBQbHVnLUluIC4gU1ZHIFZlcnNpb246IDYuMDAgQnVpbGQgMCkgIC0tPgo8c3ZnIHZlcnNpb249IjEuMSIKCSBpZD0ic3ZnMiIgaW5rc2NhcGU6ZXhwb3J0LWZpbGVuYW1lPSIvdXNyL2xvY2FsL2dvb2dsZS9ob21lL3Rob2NraW4vc3JjL2t1YmVybmV0ZXMvZG9jcy9zZXJ2aWNlc19vdmVydmlldy5wbmciIGlua3NjYXBlOmV4cG9ydC14ZHBpPSI3Ni45MTAwMDQiIGlua3NjYXBlOmV4cG9ydC15ZHBpPSI3Ni45MTAwMDQiIGlua3NjYXBlOnZlcnNpb249IjAuNDguNCByOTkzOSIgc29kaXBvZGk6ZG9jbmFtZT0ic2VydmljZXMtdXNlcnNwYWNlLW92ZXJ2aWV3LnN2ZyIgeG1sbnM6Y2M9Imh0dHA6Ly9jcmVhdGl2ZWNvbW1vbnMub3JnL25zIyIgeG1sbnM6ZGM9Imh0dHA6Ly9wdXJsLm9yZy9kYy9lbGVtZW50cy8xLjEvIiB4bWxuczppbmtzY2FwZT0iaHR0cDovL3d3dy5pbmtzY2FwZS5vcmcvbmFtZXNwYWNlcy9pbmtzY2FwZSIgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIiB4bWxuczpzb2RpcG9kaT0iaHR0cDovL3NvZGlwb2RpLnNvdXJjZWZvcmdlLm5ldC9EVEQvc29kaXBvZGktMC5kdGQiIHhtbG5zOnN2Zz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciCgkgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgeD0iMHB4IiB5PSIwcHgiIHZpZXdCb3g9IjAgMCA3NjMgNDI5IgoJIHN0eWxlPSJlbmFibGUtYmFja2dyb3VuZDpuZXcgMCAwIDc2MyA0Mjk7IiB4bWw6c3BhY2U9InByZXNlcnZlIj4KPHN0eWxlIHR5cGU9InRleHQvY3NzIj4KCS5zdDB7ZmlsbDpub25lO3N0cm9rZTojMDAwMDAwO3N0cm9rZS13aWR0aDo1LjE5NjU7fQoJLnN0MXtzdHJva2U6IzAwMDAwMDtzdHJva2Utd2lkdGg6NS4xOTY1O3N0cm9rZS1saW5lY2FwOnJvdW5kO30KCS5zdDJ7ZmlsbDpub25lO3N0cm9rZTojMDAwMDAwO3N0cm9rZS13aWR0aDo1O30KCS5zdDN7c3Ryb2tlOiMwMDAwMDA7c3Ryb2tlLXdpZHRoOjQuNzAzNjtzdHJva2UtbGluZWNhcDpyb3VuZDt9Cgkuc3Q0e2ZpbGw6bm9uZTtzdHJva2U6IzAwMDAwMDtzdHJva2Utd2lkdGg6NC4yOTI4O30KCS5zdDV7c3Ryb2tlOiMwMDAwMDA7c3Ryb2tlLXdpZHRoOjQuMjkyODtzdHJva2UtbGluZWNhcDpyb3VuZDt9Cgkuc3Q2e2ZpbGw6Izg1QkZGMTtzdHJva2U6IzAwMDAwMDtzdHJva2Utd2lkdGg6MjtzdHJva2UtbGluZWNhcDpyb3VuZDtzdHJva2UtbGluZWpvaW46cm91bmQ7fQoJLnN0N3tmb250LWZhbWlseTonQXJpYWxNVCc7fQoJLnN0OHtmb250LXNpemU6MzJweDt9Cgkuc3Q5e2ZvbnQtc2l6ZToyNHB4O30KCS5zdDEwe2ZpbGw6bm9uZTtzdHJva2U6IzAwMDAwMDtzdHJva2Utd2lkdGg6MS44ODE0O30KCS5zdDExe3N0cm9rZTojMDAwMDAwO3N0cm9rZS13aWR0aDoxLjg4MTQ7c3Ryb2tlLWxpbmVjYXA6cm91bmQ7fQoJLnN0MTJ7ZmlsbDojRjFDQjg1O3N0cm9rZTojMDAwMDAwO3N0cm9rZS13aWR0aDoyO3N0cm9rZS1saW5lY2FwOnJvdW5kO3N0cm9rZS1saW5lam9pbjpyb3VuZDt9Cgkuc3QxM3tmaWxsOiNCOUYxODU7c3Ryb2tlOiMwMDAwMDA7c3Ryb2tlLXdpZHRoOjI7c3Ryb2tlLWxpbmVjYXA6cm91bmQ7c3Ryb2tlLWxpbmVqb2luOnJvdW5kO30KCS5zdDE0e2ZpbGw6I0VEQzFGODtzdHJva2U6IzAwMDAwMDtzdHJva2Utd2lkdGg6MjtzdHJva2UtbGluZWNhcDpyb3VuZDtzdHJva2UtbGluZWpvaW46cm91bmQ7fQoJLnN0MTV7ZmlsbDojRkZFNjgwO3N0cm9rZTojMDAwMDAwO3N0cm9rZS13aWR0aDoxLjc3ODc7fQoJLnN0MTZ7ZmlsbDpub25lO3N0cm9rZTojMDAwMDAwO3N0cm9rZS13aWR0aDowLjkyMzk7fQoJLnN0MTd7Zm9udC1mYW1pbHk6J015cmlhZFByby1SZWd1bGFyJzt9Cgkuc3QxOHtmb250LXNpemU6NDBweDt9Cjwvc3R5bGU+Cjxzb2RpcG9kaTpuYW1lZHZpZXcgIGJvcmRlcmNvbG9yPSIjNjY2NjY2IiBib3JkZXJvcGFjaXR5PSIxLjAiIGlkPSJiYXNlIiBpbmtzY2FwZTpjdXJyZW50LWxheWVyPSJsYXllcjEiIGlua3NjYXBlOmN4PSIyOTEuOTI1NCIgaW5rc2NhcGU6Y3k9IjM5Mi4zMDU0NSIgaW5rc2NhcGU6ZG9jdW1lbnQtdW5pdHM9InB4IiBpbmtzY2FwZTpwYWdlb3BhY2l0eT0iMC4wIiBpbmtzY2FwZTpwYWdlc2hhZG93PSIyIiBpbmtzY2FwZTp3aW5kb3ctaGVpZ2h0PSI4MjIiIGlua3NjYXBlOndpbmRvdy1tYXhpbWl6ZWQ9IjAiIGlua3NjYXBlOndpbmRvdy13aWR0aD0iMTU1MiIgaW5rc2NhcGU6d2luZG93LXg9IjQ2IiBpbmtzY2FwZTp3aW5kb3cteT0iNDciIGlua3NjYXBlOnpvb209IjEuMDMxODM2OSIgcGFnZWNvbG9yPSIjZmZmZmZmIiBzaG93Z3JpZD0iZmFsc2UiPgoJPC9zb2RpcG9kaTpuYW1lZHZpZXc+CjxnIGlkPSJsYXllcjEiIGlua3NjYXBlOmdyb3VwbW9kZT0ibGF5ZXIiIGlua3NjYXBlOmxhYmVsPSJMYXllciAxIj4KCTxnIGlkPSJnNDE3OC0zLTgiIHRyYW5zZm9ybT0ibWF0cml4KDAsLTEsLTAuOTI1Nzg5NjIsMCw5MzYuNDQ0MTMsMTAyOS4yNjg2KSI+CgkJPHBhdGggaWQ9InBhdGg0MTc0LTMtNCIgaW5rc2NhcGU6Y29ubmVjdG9yLWN1cnZhdHVyZT0iMCIgY2xhc3M9InN0MCIgZD0iTTgxMy4xLDc0Ni4xYzAtNzEuMywwLTcxLjMsMC03MS4zIi8+CgkJCgkJCTxwYXRoIGlkPSJwYXRoNDE3Ni05LTAiIGlua3NjYXBlOmZsYXRzaWRlZD0idHJ1ZSIgaW5rc2NhcGU6cmFuZG9taXplZD0iMCIgaW5rc2NhcGU6cm91bmRlZD0iMCIgaW5rc2NhcGU6dHJhbnNmb3JtLWNlbnRlci15PSItMi41IiBzb2RpcG9kaTphcmcxPSIyLjYxNzk5MzkiIHNvZGlwb2RpOmFyZzI9IjMuNjY1MTkxNCIgc29kaXBvZGk6Y3g9IjMwOC44NTcxNSIgc29kaXBvZGk6Y3k9Ijc1My43OTA3NyIgc29kaXBvZGk6cjE9IjEwIiBzb2RpcG9kaTpyMj0iNSIgc29kaXBvZGk6c2lkZXM9IjMiIHNvZGlwb2RpOnR5cGU9InN0YXIiIGNsYXNzPSJzdDEiIGQ9IgoJCQlNODA0LjgsNjg0LjFsOC43LTE1bDguNywxNUg4MDQuOHoiLz4KCTwvZz4KCTxnIGlkPSJnNDMyNCIgdHJhbnNmb3JtPSJtYXRyaXgoMC45NjU5MjU4MywwLjI1ODgxOTA1LDAuMjU4ODE5MDUsLTAuOTY1OTI1ODMsLTM4LjgxMDc0NCwxMDc2LjIzOCkiPgoJCTxwYXRoIGlkPSJwYXRoNDE3NC0zLTIiIGlua3NjYXBlOmNvbm5lY3Rvci1jdXJ2YXR1cmU9IjAiIGNsYXNzPSJzdDIiIGQ9Ik0yMjcuOSw5NTkuN2MtMTAyLTExMy44LTEwMi0xMTMuOC0xMDItMTEzLjgiLz4KCQkKCQkJPHBhdGggaWQ9InBhdGg0MTc2LTktOSIgaW5rc2NhcGU6ZmxhdHNpZGVkPSJ0cnVlIiBpbmtzY2FwZTpyYW5kb21pemVkPSIwIiBpbmtzY2FwZTpyb3VuZGVkPSIwIiBpbmtzY2FwZTp0cmFuc2Zvcm0tY2VudGVyLXk9Ii0yLjUiIHNvZGlwb2RpOmFyZzE9IjIuNjE3OTkzOSIgc29kaXBvZGk6YXJnMj0iMy42NjUxOTE0IiBzb2RpcG9kaTpjeD0iMzA4Ljg1NzE1IiBzb2RpcG9kaTpjeT0iNzUzLjc5MDc3IiBzb2RpcG9kaTpyMT0iMTAiIHNvZGlwb2RpOnIyPSI1IiBzb2RpcG9kaTpzaWRlcz0iMyIgc29kaXBvZGk6dHlwZT0ic3RhciIgY2xhc3M9InN0MyIgZD0iCgkJCU0xMTguOCw4NTIuOGwtNC44LTE4LjRsMTcuOCw2LjlMMTE4LjgsODUyLjh6Ii8+Cgk8L2c+Cgk8ZyBpZD0iZzQzMjQtOCIgdHJhbnNmb3JtPSJtYXRyaXgoLTAuOTY1OTI1ODMsMC4yNTg4MTkwNSwtMC4yNTg4MTkwNSwtMC45NjU5MjU4Myw4ODguMzI5NjQsMTA3Ni4yMzgpIj4KCQk8cGF0aCBpZD0icGF0aDQxNzQtMy0yLTciIGlua3NjYXBlOmNvbm5lY3Rvci1jdXJ2YXR1cmU9IjAiIGNsYXNzPSJzdDIiIGQ9Ik0yMDYuNiw5NTRjLTEwMi0xMTMuOC0xMDItMTEzLjgtMTAyLTExMy44Ii8+CgkJCgkJCTxwYXRoIGlkPSJwYXRoNDE3Ni05LTktMyIgaW5rc2NhcGU6ZmxhdHNpZGVkPSJ0cnVlIiBpbmtzY2FwZTpyYW5kb21pemVkPSIwIiBpbmtzY2FwZTpyb3VuZGVkPSIwIiBpbmtzY2FwZTp0cmFuc2Zvcm0tY2VudGVyLXk9Ii0yLjUiIHNvZGlwb2RpOmFyZzE9IjIuNjE3OTkzOSIgc29kaXBvZGk6YXJnMj0iMy42NjUxOTE0IiBzb2RpcG9kaTpjeD0iMzA4Ljg1NzE1IiBzb2RpcG9kaTpjeT0iNzUzLjc5MDc3IiBzb2RpcG9kaTpyMT0iMTAiIHNvZGlwb2RpOnIyPSI1IiBzb2RpcG9kaTpzaWRlcz0iMyIgc29kaXBvZGk6dHlwZT0ic3RhciIgY2xhc3M9InN0MyIgZD0iCgkJCU05Ny42LDg0Ny4xbC00LjgtMTguNGwxNy44LDYuOUw5Ny42LDg0Ny4xeiIvPgoJPC9nPgoJPGcgaWQ9Imc0MTc4LTMtOSIgdHJhbnNmb3JtPSJtYXRyaXgoMSwwLDAsLTEuMzU2NjA2Niw3OS4yNDAwMTQsMTY5OS41NDMxKSI+CgkJPHBhdGggaWQ9InBhdGg0MTc0LTMtOCIgaW5rc2NhcGU6Y29ubmVjdG9yLWN1cnZhdHVyZT0iMCIgY2xhc3M9InN0NCIgZD0iTTM0OC4xLDExMDguOGMwLTcxLjMsMC03MS4zLDAtNzEuMyIvPgoJCQoJCQk8cGF0aCBpZD0icGF0aDQxNzYtOS01IiBpbmtzY2FwZTpmbGF0c2lkZWQ9InRydWUiIGlua3NjYXBlOnJhbmRvbWl6ZWQ9IjAiIGlua3NjYXBlOnJvdW5kZWQ9IjAiIGlua3NjYXBlOnRyYW5zZm9ybS1jZW50ZXIteT0iLTIuNSIgc29kaXBvZGk6YXJnMT0iMi42MTc5OTM5IiBzb2RpcG9kaTphcmcyPSIzLjY2NTE5MTQiIHNvZGlwb2RpOmN4PSIzMDguODU3MTUiIHNvZGlwb2RpOmN5PSI3NTMuNzkwNzciIHNvZGlwb2RpOnIxPSIxMCIgc29kaXBvZGk6cjI9IjUiIHNvZGlwb2RpOnNpZGVzPSIzIiBzb2RpcG9kaTp0eXBlPSJzdGFyIiBjbGFzcz0ic3Q1IiBkPSIKCQkJTTMzOS44LDEwNDYuOGw4LjctMTVsOC43LDE1SDMzOS44eiIvPgoJPC9nPgoJPGcgaWQ9ImczOTM3IiB0cmFuc2Zvcm09InRyYW5zbGF0ZSgyMS42NDM1NDQsNzE5LjczMDc0KSI+CgkJPGcgaWQ9ImczODY4IiB0cmFuc2Zvcm09Im1hdHJpeCgwLjg4NzkyMzM3LDAsMCwxLDQzLjUwOTc1LDYuNTI1MDAwMWUtNikiPgoJCQk8cmVjdCBpZD0icmVjdDI5ODUiIHg9IjQyLjQiIHk9Ii00MTUuMSIgY2xhc3M9InN0NiIgd2lkdGg9IjIyNC4zIiBoZWlnaHQ9IjExOC42Ii8+CgkJCTxnIGlkPSJnMzg2MSI+CgkJCQk8dGV4dCB0cmFuc2Zvcm09Im1hdHJpeCgxIDAgMCAxIDQ5LjA5OTEgLTM4NC4xNTQ1KSIgY2xhc3M9InN0NyBzdDgiPkJhY2tlbmQgUG9kIDE8L3RleHQ+CgkJCQk8dGV4dCB0cmFuc2Zvcm09Im1hdHJpeCgxIDAgMCAxIDQ5LjUzMTMgLTM0NS4wNjY3KSIgY2xhc3M9InN0NyBzdDkiPmxhYmVsczogYXBwPU15QXBwPC90ZXh0PgoJCQkJPHRleHQgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSA0OS41MzEzIC0zMTUuMDY2NykiIGNsYXNzPSJzdDcgc3Q5Ij5wb3J0OiA5Mzc2PC90ZXh0PgoJCQk8L2c+CgkJPC9nPgoJCTxnIGlkPSJnMzg2OC03IiB0cmFuc2Zvcm09Im1hdHJpeCgwLjg4NzkyMzM3LDAsMCwxLDI2Mi4wMDIzMSw2LjUyNTAwMDFlLTYpIj4KCQkJPHJlY3QgaWQ9InJlY3QyOTg1LTEiIHg9IjQyLjQiIHk9Ii00MTUuMSIgY2xhc3M9InN0NiIgd2lkdGg9IjIyNC4zIiBoZWlnaHQ9IjExOC42Ii8+CgkJCTxnIGlkPSJnMzg2MS05Ij4KCQkJCTx0ZXh0IHRyYW5zZm9ybT0ibWF0cml4KDEgMCAwIDEgNDkuMDk5MiAtMzg0LjE1NDUpIiBjbGFzcz0ic3Q3IHN0OCI+QmFja2VuZCBQb2QgMjwvdGV4dD4KCQkJCTx0ZXh0IHRyYW5zZm9ybT0ibWF0cml4KDEgMCAwIDEgNDkuNTMxNSAtMzQ1LjA2NjcpIiBjbGFzcz0ic3Q3IHN0OSI+bGFiZWxzOiBhcHA9TXlBcHA8L3RleHQ+CgkJCQk8dGV4dCB0cmFuc2Zvcm09Im1hdHJpeCgxIDAgMCAxIDQ5LjUzMTUgLTMxNS4wNjY3KSIgY2xhc3M9InN0NyBzdDkiPnBvcnQ6IDkzNzY8L3RleHQ+CgkJCTwvZz4KCQk8L2c+CgkJPGcgaWQ9ImczODY4LTMiIHRyYW5zZm9ybT0ibWF0cml4KDAuODg3OTIzMzcsMCwwLDEsNDgwLjQ5NDg5LDYuNTI1MDAwMWUtNikiPgoJCQk8cmVjdCBpZD0icmVjdDI5ODUtMiIgeD0iNDIuNCIgeT0iLTQxNS4xIiBjbGFzcz0ic3Q2IiB3aWR0aD0iMjI0LjMiIGhlaWdodD0iMTE4LjYiLz4KCQkJPGcgaWQ9ImczODYxLTMiPgoJCQkJPHRleHQgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSA0OS4wOTg4IC0zODQuMTU0NSkiIGNsYXNzPSJzdDcgc3Q4Ij5CYWNrZW5kIFBvZCAzPC90ZXh0PgoJCQkJPHRleHQgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSA0OS41MzEgLTM0NS4wNjY3KSIgY2xhc3M9InN0NyBzdDkiPmxhYmVsczogYXBwPU15QXBwPC90ZXh0PgoJCQkJPHRleHQgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSA0OS41MzEgLTMxNS4wNjY3KSIgY2xhc3M9InN0NyBzdDkiPnBvcnQ6IDkzNzY8L3RleHQ+CgkJCTwvZz4KCQk8L2c+Cgk8L2c+Cgk8ZyBpZD0iZzQxNzgtMy00IiB0cmFuc2Zvcm09Im1hdHJpeCgtMC44MzA1MjQ5LC0wLjU1Njk4MTUsMC42MjkzOTMzMiwtMC45Mzg0OTk0NSwzNjUuNTQ4NTUsMTQ4Ny44Mzk2KSI+CgkJPHBhdGggaWQ9InBhdGg0MTc0LTMtOSIgaW5rc2NhcGU6Y29ubmVjdG9yLWN1cnZhdHVyZT0iMCIgY2xhc3M9InN0MTAiIGQ9Ik01OTMuMSwxMTEzLjJjMC03MS4zLDAtNzEuMywwLTcxLjMiLz4KCQkKCQkJPHBhdGggaWQ9InBhdGg0MTc2LTktMSIgaW5rc2NhcGU6ZmxhdHNpZGVkPSJ0cnVlIiBpbmtzY2FwZTpyYW5kb21pemVkPSIwIiBpbmtzY2FwZTpyb3VuZGVkPSIwIiBpbmtzY2FwZTp0cmFuc2Zvcm0tY2VudGVyLXk9Ii0yLjUiIHNvZGlwb2RpOmFyZzE9IjIuNjE3OTkzOSIgc29kaXBvZGk6YXJnMj0iMy42NjUxOTE0IiBzb2RpcG9kaTpjeD0iMzA4Ljg1NzE1IiBzb2RpcG9kaTpjeT0iNzUzLjc5MDc3IiBzb2RpcG9kaTpyMT0iMTAiIHNvZGlwb2RpOnIyPSI1IiBzb2RpcG9kaTpzaWRlcz0iMyIgc29kaXBvZGk6dHlwZT0ic3RhciIgY2xhc3M9InN0MTEiIGQ9IgoJCQlNNTg0LjgsMTA1MS4ybDguNy0xNWw4LjcsMTVMNTg0LjgsMTA1MS4yeiIvPgoJPC9nPgoJPGcgaWQ9Imc0MTc4LTMiIHRyYW5zZm9ybT0ibWF0cml4KDEsMCwwLC0wLjkyNTc4OTYyLC0xNzAuOTgxMzYsMTI2OC43Njk5KSI+CgkJPHBhdGggaWQ9InBhdGg0MTc0LTMiIGlua3NjYXBlOmNvbm5lY3Rvci1jdXJ2YXR1cmU9IjAiIGNsYXNzPSJzdDAiIGQ9Ik0zNDguMSwxMjcyLjFjMC03MS4zLDAtNzEuMywwLTcxLjMiLz4KCQkKCQkJPHBhdGggaWQ9InBhdGg0MTc2LTkiIGlua3NjYXBlOmZsYXRzaWRlZD0idHJ1ZSIgaW5rc2NhcGU6cmFuZG9taXplZD0iMCIgaW5rc2NhcGU6cm91bmRlZD0iMCIgaW5rc2NhcGU6dHJhbnNmb3JtLWNlbnRlci15PSItMi41IiBzb2RpcG9kaTphcmcxPSIyLjYxNzk5MzkiIHNvZGlwb2RpOmFyZzI9IjMuNjY1MTkxNCIgc29kaXBvZGk6Y3g9IjMwOC44NTcxNSIgc29kaXBvZGk6Y3k9Ijc1My43OTA3NyIgc29kaXBvZGk6cjE9IjEwIiBzb2RpcG9kaTpyMj0iNSIgc29kaXBvZGk6c2lkZXM9IjMiIHNvZGlwb2RpOnR5cGU9InN0YXIiIGNsYXNzPSJzdDEiIGQ9IgoJCQlNMzM5LjgsMTIxMC4xbDguNy0xNWw4LjcsMTVIMzM5Ljh6Ii8+Cgk8L2c+Cgk8ZyBpZD0iZzQwOTAiIHRyYW5zZm9ybT0ibWF0cml4KDAuODkwNjcwMDMsMCwwLDEsLTEzMC45NzI5NSwtMTcyLjM2Mjg2KSI+CgkJPHJlY3QgaWQ9InJlY3QyOTg1LTQiIHg9IjIzNC4xIiB5PSIyMjguNSIgY2xhc3M9InN0MTIiIHdpZHRoPSIyMjQuMyIgaGVpZ2h0PSI1OC42Ii8+CgkJPGcgaWQ9ImczODYxLTYiIHRyYW5zZm9ybT0idHJhbnNsYXRlKDIxNy42MTc3LDY1Mi44MjUxNikiPgoJCQk8dGV4dCB0cmFuc2Zvcm09Im1hdHJpeCgxIDAgMCAxIDc5LjkyNTEgLTM4NC4yMzQ3KSIgY2xhc3M9InN0NyBzdDgiPkNsaWVudCA8L3RleHQ+CgkJPC9nPgoJPC9nPgoJPGcgaWQ9Imc0MTY4IiB0cmFuc2Zvcm09Im1hdHJpeCgwLjg5MDY3MDAzLDAsMCwxLDI2My42NTkyMiw3NC4yMDU0NzMpIj4KCQk8cmVjdCBpZD0icmVjdDI5ODUtNC0wIiB4PSI2My4xIiB5PSIxMTIuOCIgY2xhc3M9InN0MTMiIHdpZHRoPSIyNTAiIGhlaWdodD0iNTguNiIvPgoJCTxnIGlkPSJnMzg2MS02LTIiIHRyYW5zZm9ybT0idHJhbnNsYXRlKDM0Ljc0NzQzMyw1MzQuMjYyODcpIj4KCQkJPHRleHQgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSA3Mi44MDUyIC0zODMuNzg2NykiIGNsYXNzPSJzdDcgc3Q4Ij5rdWJlLXByb3h5PC90ZXh0PgoJCTwvZz4KCTwvZz4KCTxnIGlkPSJnNDE2OC01IiB0cmFuc2Zvcm09InRyYW5zbGF0ZSg0NzguODIzMzYsMjcuMjkxOTY1KSI+CgkJPGcgaWQ9Imc0MjM4IiB0cmFuc2Zvcm09InRyYW5zbGF0ZSgyMi4wODc0MjksLTg2LjM0MTc3KSI+CgkJCTxyZWN0IGlkPSJyZWN0Mjk4NS00LTAtNiIgeD0iNjIuOSIgeT0iMTEyLjgiIGNsYXNzPSJzdDE0IiB3aWR0aD0iMTkxLjgiIGhlaWdodD0iNTguNiIvPgoJCQk8ZyBpZD0iZzM4NjEtNi0yLTYiIHRyYW5zZm9ybT0idHJhbnNsYXRlKDM5LjEwNzQyOSw1MzQuMjYyODcpIj4KCQkJCTx0ZXh0IHRyYW5zZm9ybT0ibWF0cml4KDEgMCAwIDEgNDcuNzEwOCAtMzg0LjE1NDQpIiBjbGFzcz0ic3Q3IHN0OCI+YXBpc2VydmVyPC90ZXh0PgoJCQk8L2c+CgkJPC9nPgoJPC9nPgoJPHBhdGggaWQ9InBhdGgzODg0IiBpbmtzY2FwZTpjb25uZWN0b3ItY3VydmF0dXJlPSIwIiBjbGFzcz0ic3QxNSIgZD0iTTEyOS42LDE1My4xYy0xNC41LDAtMjcsOC42LTMzLDIxLjIKCQljLTUtMC45LTEwLjMtMS40LTE1LjgtMS40Yy0zNC4zLDAtNjIuMiwxOS4xLTYyLjIsNDIuNnMyNy44LDQyLjYsNjIuMiw0Mi42YzE2LjksMCwzMi4yLTQuNiw0My40LTEyLjJjNy40LDE2LjMsMzQsMjguMyw2NS43LDI4LjMKCQljMzMuNSwwLDYxLjQtMTMuNCw2Ni44LTMxLjFjMTctNS4zLDI4LjYtMTUuNywyOC42LTI3LjdjMC0xNy4zLTI0LjQtMzEuNC01NC40LTMxLjRjLTguNSwwLTE2LjUsMS4xLTIzLjYsMy4xCgkJYy0yLTEwLjUtMTQuOC0xOC42LTMwLjMtMTguNmMtNS44LDAtMTEuMiwxLjItMTUuOSwzLjFDMTU0LjgsMTYwLjYsMTQzLDE1My4xLDEyOS42LDE1My4xTDEyOS42LDE1My4xeiIvPgoJPGcgaWQ9ImczODYxLTYtMjgiIHRyYW5zZm9ybT0ibWF0cml4KDAuODkwNjcwMDMsMCwwLDEsLTMxLjA5MTgzNiw1ODcuNjc5MDQpIj4KCQk8dGV4dCB0cmFuc2Zvcm09Im1hdHJpeCgxIDAgMCAxIDEwNi4wMzk2IC0zODYuMTczMikiIGNsYXNzPSJzdDcgc3Q4Ij5TZXJ2aWNlSVA8L3RleHQ+CgkJPHRleHQgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSAxMDYuMDM5NiAtMzQ2LjE3MzIpIiBjbGFzcz0ic3Q3IHN0OCI+KGlwdGFibGVzKSA8L3RleHQ+Cgk8L2c+Cgk8cmVjdCBpZD0icmVjdDM4ODkiIHg9IjcuMSIgeT0iOC40IiBjbGFzcz0ic3QxNiIgd2lkdGg9IjU0NC43IiBoZWlnaHQ9IjI2Ny42Ii8+Cgk8dGV4dCB0cmFuc2Zvcm09Im1hdHJpeCgxIDAgMCAxIDExLjk2OTEgNDUuMjcwNSkiIGNsYXNzPSJzdDE3IHN0MTgiPk5vZGU8L3RleHQ+CjwvZz4KPC9zdmc+Cg==)

  **userspace **모드로 설정하면 위의 그림과 같은 구조를 가지게 된다. 클라이언트에서 서비스IP를 통해서 요청을 하게 되면 **iptables**를 거쳐서 큐브프록시가 요청을 받은 다음에 그 서비스IP가 연결되어야 하는 적절한 포드로 연결을 해준다. 이때 요청을 포드들에 나눠주는 방식은 라운드 로빈이다.

> 라운드로빈 : 시분할 시스템을 위해 설계된 선점형 스케줄링의 하나로서, 프로세스들 사이에 우선순위를 두지 않고 순서대로 시간단위로 CPU를 할당하는 방식의 CPU 스케줄링 알고리즘이다.





## iptables 모드

![](data:image/png;base64,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)

  **iptables** 모드는 위의 그림과 같은 구조를 갖는다. **userspace** 모드와 다른 점은 큐브 프록시는 **iptable**을 관리하는 역할만 하고 직접 클라이언트로부터 트래픽을 받지 않는다. 클라이언트로부터 오는 모든 요청은 **iptables**를 거쳐서 직접 포드로 전달된다. 그래서 **userspace** 모드보다 빠른 성능을 갖는다. **userspace** 모드에서는 포드 하나로의 요청이 실패하면 자동으로 다른 포드로 연결을 재시도하지만 **iptables** 모드에서는 포드 하나로 가는 요청이 실패하면 재시도를 하지 않고 그냥 요청이 실패한다. 이런걸 방지하기 위해서 **readiness Probe** 설정이 잘 되어있어야 한다.

> readiness Probe(준비성 프로브)) : 컨테이너가 요청을 처리할 준비가 되어있는지 여부를 나타낸다. 





## ipvs 모드

![](data:image/png;base64,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)

  **ipvs**(IP Virtual Server) 모드는 리눅스 커널에 있는 L4 로드밸런싱 기술로 Netfilter에 포함되어 있다. 그래서 IPVS 커널모듈이 설치되어 있어야 한다. **ipvs**는 커널스페이스에서 작동하고 데이터 구조를 해시테이블로 저장해서 가지고 있기 때문에 **iptables**보다 빠르고 좋은 성능을 낸다. 그리고 **ipvs**는 더 많은 로드밸런싱 알고리즘을 가지고 있어서 이걸 이용할 수 있다. rr(round-robin), lc(least connection), dh(destination hashing), sh(source hasing), sed(shortest expected delay), nq(never queue) 등의 다양한 로드밸런싱 알고리즘을 제공해준다.






