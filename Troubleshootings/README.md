---
title: Troubeshooting
author: Dongook Son
---

# JupyterHub with Kubernetes for Yonsei Data Science Machine

## Docker troubleshooting

### bind: address already in use
컨테이너를 start, restart할 때 발생 (docker 가 정상적으로 종료되지 않아 발생한 에러인 듯)

- Error message:
  
```
Error response from daemon: driver failed programming external connectivity on endpoint freeipa (3cdc3391a606f09173f4bc2538e19e12f7ca7c4fef35ddc97c27315872b1f908): Error starting userland proxy: **listen tcp 0.0.0.0:9389: bind: address already in use**
```

포트가 이미 할당되어 있어서 발생한 에러로 해당 포트를 사용 중인지 확인. -i 뒤에 한 칸 띄우고, :랑 포트번호는 붙여 써야 함

```sudo lsof -i :8000```

output:
```
COMMAND PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   989     root    9u  IPv4  21377      0t0  TCP *:2224 (LISTEN)
nginx   990 www-data    9u  IPv4  21377      0t0  TCP *:2224 (LISTEN)
nginx   991 www-data    9u  IPv4  21377      0t0  TCP *:2224 (LISTEN)
```

sudo kill -9로 할당되어 있는 포트를 죽인 후 다시 docker 컨테이너를 재시작하면 문제 해결

```
sudo kill -9 989
sudo kill -9 990
sudo kill -9 991```

[^fn1]: https://github.com/jupyterhub/jupyterhub
[^fn2]: https://www.rstudio.com/products/workbench/
[^fn3]: https://jupyterhub.readthedocs.io/en/stable/
[^fn4]: https://zero-to-jupyterhub.readthedocs.io/en/latest/
[^fn5]: https://www.tornadoweb.org/en/stable/
[^fn6]: https://www.npmjs.com/package/configurable-http-proxy
[^fn7]: https://jupyter.utoronto.ca/hub/login?next=%2Fhub%2F
