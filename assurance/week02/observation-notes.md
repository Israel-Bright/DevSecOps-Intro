# Lab startup observations

Recorded September 16, 2026. These are initial observations, not a completed threat model.

## Completed

- Opened Docker Desktop and confirmed both Docker Client and Server respond.
- Started container `juice` using `bkimminich/juice-shop:v20.0.0`.
- Bound the application to `127.0.0.1:3000` with the assignment command:

  ```sh
  docker run -d --name juice -p 127.0.0.1:3000:3000 bkimminich/juice-shop:v20.0.0
  ```

- Downloaded image digest: `sha256:fd58bdc9745416afce8184ee0666278a436574633ea7880365153a63bfd418b0`.
- Opened the HTML worksheet and `http://localhost:3000` in the default browser.

## First evidence

Command:

```sh
curl --max-time 15 -sS -D - -o /dev/null http://localhost:3000/
```

Response observed at `2026-09-16 22:46:41 GMT`:

```text
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
Content-Type: text/html; charset=UTF-8
```

The complete response-header output contained no `Content-Security-Policy` or `Content-Security-Policy-Report-Only` header. The excerpt above lists selected returned headers.

This supports only the observation that the root response checked at that time did not send those headers. It does not establish that other routes lack them, that no HTML policy exists, or that script injection or account compromise is possible. No exploitation was performed.

## Next steps

1. Repository setup is complete: `https://github.com/Israel-Bright/DevSecOps-Intro` is cloned into the materials folder as `DevSecOps-Intro/`. Save assignment outputs in this clone's `assurance/week02/` directory. Push authentication has not yet been verified.
2. In the application, register a lab account, search, and open an item; record what you observe.
3. In worksheet step 2, download a session backup. Opening the page alone does not complete or save your answers.
4. Continue with the model and analysis using `README.md`.
5. Declare AI assistance accurately: Docker setup, initial header inspection, and preparation of guidance/notes have been assisted.

The container is left running for your work. When finished, use `docker rm -f juice` as required by the lab.
