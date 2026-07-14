# Shell (curl) example

Requires `curl` and `jq`.

```bash
export CHECKNUMBER_API_KEY="YOUR_API_KEY"
./phone_checker.sh numbers.txt   # one E.164 number per line
```

Submits to `POST /v1/tasks` (`task_type=phoneCheck`), polls `POST /v1/gettasks`, downloads results. Full docs: https://docs.checknumber.ai/phone-number-validation
