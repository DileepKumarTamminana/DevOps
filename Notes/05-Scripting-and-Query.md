# 05 — Scripting & Query

Covers: Bash · PowerShell · SQL

Scripting is how DevOps engineers **automate** repetitive work. SQL is how you query data.

---

## 1. Bash (Linux shell scripting)

### Anatomy of a script
```bash
#!/bin/bash            # "shebang" — tells the OS to run with bash
set -euo pipefail      # safety: exit on error, unset vars, failed pipes

echo "Hello, $USER"
```
Run with `chmod +x script.sh && ./script.sh`.

### Variables
```bash
name="Dileep"          # NO spaces around =
echo "Hi $name"        # use $ to read; double quotes allow expansion
count=$(ls | wc -l)    # command substitution — capture output
readonly PI=3.14       # constant
```

### Conditionals
```bash
if [ "$count" -gt 10 ]; then
  echo "many files"
elif [ "$count" -eq 0 ]; then
  echo "empty"
else
  echo "few files"
fi
```
Common test operators: `-eq -ne -gt -lt -ge -le` (numbers); `=`, `!=` (strings);
`-f file` (exists & is file), `-d dir`, `-z str` (empty), `-n str` (non-empty).

### Loops
```bash
for f in *.log; do
  echo "Processing $f"
done

while read -r line; do
  echo "$line"
done < input.txt
```

### Functions & arguments
```bash
deploy() {
  local env="$1"            # first argument
  echo "Deploying to $env"
}
deploy "production"
```
Script args: `$1 $2 …`, `$@` (all), `$#` (count), `$0` (script name).

### Exit codes & error handling
```bash
command || { echo "failed"; exit 1; }   # run fallback if command fails
echo $?                                  # exit code of last command (0 = success)
```

### Text-processing toolkit (the DevOps superpower)
```bash
grep "ERROR" app.log              # find lines
grep -c "ERROR" app.log           # count matches
awk '{print $1}' file             # print first column
awk -F',' '{print $2}' data.csv   # use comma delimiter
sed 's/old/new/g' file            # find & replace
cut -d':' -f1 /etc/passwd         # cut by delimiter
sort | uniq -c | sort -nr         # frequency count
xargs                             # build commands from input
```
Example one-liner — top 5 IPs in an access log:
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -5
```

> **Interview-ready:** "I start scripts with `set -euo pipefail` for safety, use command
> substitution `$(...)` to capture output, and combine `grep`/`awk`/`sed` for log analysis.
> Exit code 0 means success."

---

## 2. PowerShell (Windows automation)

PowerShell is object-based (commands pass **objects**, not plain text), which makes it
powerful for Windows/Azure automation.

### Cmdlets follow Verb-Noun
```powershell
Get-Process
Get-Service
Get-ChildItem            # like 'ls' / 'dir'
Set-Location C:\Temp     # like 'cd'
```

### Variables & types
```powershell
$name = "Dileep"
$count = (Get-ChildItem).Count
Write-Host "Found $count items"
$list = @(1, 2, 3)        # array
$map  = @{ env = "prod"; region = "eastus" }   # hashtable
```

### The pipeline (objects, not text)
```powershell
Get-Process | Where-Object { $_.CPU -gt 100 } | Sort-Object CPU -Descending | Select-Object -First 5
```
- `$_` = the current object in the pipeline.
- `Where-Object` filters, `Select-Object` picks properties, `Sort-Object` sorts,
  `ForEach-Object` loops.

### Conditionals & loops
```powershell
if ($count -gt 10) { "many" } elseif ($count -eq 0) { "empty" } else { "few" }

foreach ($f in Get-ChildItem *.log) {
  Write-Host "Processing $($f.Name)"
}
```
Comparison operators are words: `-eq -ne -gt -lt -ge -le -like -match`.

### Functions
```powershell
function Deploy-App {
  param([string]$Environment)
  Write-Host "Deploying to $Environment"
}
Deploy-App -Environment "production"
```

### Useful for DevOps
```powershell
Invoke-RestMethod -Uri "https://api.site.com/health"   # call an API
Get-Content app.log -Tail 50 -Wait                      # like 'tail -f'
Select-String -Path *.log -Pattern "ERROR"              # like grep
Get-Service "W3SVC" | Restart-Service                   # manage services
# Azure: the Az PowerShell module
Connect-AzAccount; Get-AzResourceGroup
```

> **Interview-ready:** "PowerShell passes objects down the pipeline, so I can filter on
> properties with `Where-Object` and access them with `$_`. I use it for Windows admin,
> calling REST APIs, and Azure automation via the Az module."

### Bash ↔ PowerShell quick map
| Task | Bash | PowerShell |
|---|---|---|
| List files | `ls` | `Get-ChildItem` |
| Print text | `echo` | `Write-Host` / `Write-Output` |
| Find in files | `grep` | `Select-String` |
| Follow log | `tail -f` | `Get-Content -Wait -Tail` |
| Variable | `$x=1` | `$x = 1` |
| Pipe filter | `\| grep x` | `\| Where-Object {...}` |

---

## 3. SQL (querying data)

SQL queries **relational databases** (tables with rows & columns).

### The core query — SELECT
```sql
SELECT first_name, last_name        -- columns (use * for all)
FROM   employees                    -- table
WHERE  department = 'IT'            -- filter rows
ORDER BY last_name ASC              -- sort
LIMIT 10;                           -- (SQL Server: TOP 10)
```

### Logical order of execution (not the written order!)
`FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`
Knowing this explains why you can't use a `SELECT` alias in `WHERE`.

### Filtering
```sql
WHERE age > 30
WHERE name LIKE 'A%'              -- starts with A
WHERE status IN ('open','pending')
WHERE created BETWEEN '2024-01-01' AND '2024-12-31'
WHERE manager_id IS NULL
```

### Aggregation & grouping
```sql
SELECT department, COUNT(*) AS headcount, AVG(salary) AS avg_pay
FROM   employees
GROUP BY department
HAVING COUNT(*) > 5;             -- filter on aggregates (WHERE can't)
```
Aggregate functions: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.

### JOINs (combine tables) — very common interview topic
```sql
SELECT e.name, d.dept_name
FROM   employees e
JOIN   departments d ON e.dept_id = d.id;
```
| JOIN type | Returns |
|---|---|
| **INNER JOIN** | Only rows matching in both tables |
| **LEFT JOIN** | All left rows + matches (NULLs if none on right) |
| **RIGHT JOIN** | All right rows + matches |
| **FULL OUTER JOIN** | All rows from both sides |

### Modifying data (DML)
```sql
INSERT INTO employees (name, dept_id) VALUES ('Asha', 3);
UPDATE employees SET salary = 60000 WHERE id = 12;
DELETE FROM employees WHERE id = 12;
```
> **Always** use a `WHERE` clause with UPDATE/DELETE — without it you change *every* row.

### Statement categories
- **DDL** (structure): `CREATE`, `ALTER`, `DROP`
- **DML** (data): `INSERT`, `UPDATE`, `DELETE`
- **DQL** (query): `SELECT`
- **DCL** (permissions): `GRANT`, `REVOKE`
- **TCL** (transactions): `COMMIT`, `ROLLBACK`

> **Interview-ready:** "SELECT/FROM/WHERE/GROUP BY/HAVING/ORDER BY is the structure.
> INNER JOIN returns only matches; LEFT JOIN keeps all left rows. HAVING filters aggregates,
> WHERE filters rows. I never run UPDATE/DELETE without a WHERE."

---

## Practice checklist
- [ ] Write a Bash script that loops over `.log` files and counts ERROR lines.
- [ ] Write the top-5-IPs one-liner from memory.
- [ ] Use a PowerShell pipeline to find the top 5 CPU-consuming processes.
- [ ] Write a SQL query with a GROUP BY + HAVING.
- [ ] Explain INNER vs LEFT JOIN with a diagram.
