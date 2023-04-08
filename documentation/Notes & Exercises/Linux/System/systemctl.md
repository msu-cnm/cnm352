## systemctl

Show all available services

```bash
systemctl list-unit-files
```

Don't paginate the logs on the status output:

```bash
sudo systemctl --no-pager status SERVICENAME
```

