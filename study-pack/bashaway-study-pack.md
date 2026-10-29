# SLIIT Bashaway Study Pack (2022–2024 editions), for preparing for Bashaway 2026

This pack is a condensed copy of the `sliit-foss/bashaway-challenges` repo. For **every** challenge it keeps the question, the starter code and the test file. Repeated test boilerplate is cut down, and the full version is shown once under "Common test harness" below. Solutions are included **only where they teach something**. The trivial ones are reduced to a one-line *Key idea*.

How to use it with Claude: upload this file, then ask for things like *"Quiz me on 2024 Stampede without showing the solution"*, *"Explain the jq in Dot Detective step by step"* or *"Give me 5 new practice challenges in the style of the 2024 medium ones"*.

## How Bashaway challenges work (based on the tests)

- You write **one file, `execute.sh`**, and the grader runs `bash execute.sh [args]` from the challenge root with Jest. Unless a challenge says *Constraints: None*, it must stay **pure bash**. The tests search the script for words like `node`, `.js`, `python` and `.py`. They also fail if any other `.sh` file exists outside `src/`, or if extra npm packages are installed.
- **Inputs in `src/` are regenerated randomly by the test**, so never hardcode file names, counts or values. **Outputs go to `out/`**, which the test usually deletes first, so always `mkdir -p out`.
- **"Should not crash"** means: when an argument, file or directory is missing, print a message and `exit 0`.
- **Character limits** count the whole file, including newlines and the shebang. Leave out `#!/bin/bash` when golfing.
- The grader runs on **GitHub Actions Ubuntu runners**. You get `sudo`, `apt-get`, `docker`, `jq`, `curl` and `openssl`, plus Actions extras: append `KEY=val` to `$GITHUB_ENV` to export env vars to the test step, and `$GITHUB_API_URL` is set.
- Long-running things (servers, producers, port-forwards) have to go to the **background** (`&`, `nohup`, `disown`) so the script can exit. Wait until the service is ready (`sleep`, or a retry loop with `nc -z`, `curl` or `kubectl wait`) before the script ends.
- **Read the test first.** It is the real spec. It often shows exact file names, ports, credentials and loopholes the question doesn't mention (see *Fleet-Footed*, *access-denied*, *Ancient Records* and *exclusion*).
- Difficulty trend: 2023 was mostly text processing, Docker and git. 2024 added Kubernetes (minikube/k3d, metrics-server, Helm, sealed-secrets, Ingress), CI workflows run locally with `act`, TLS, load balancing and rate limiting. Expect 2026 to go further toward cloud-native and DevOps topics.

## Skills map (challenge → main skill)

| Skill | Challenges |
|---|---|
| Text processing: grep / sed / awk / tr / paste / fold / sort | clay-tablets, eldoria, log-cleaner, fate-of-atlantis, numerological-incantation, Gateway of Argentum, Operation Renard, Graceful Presentation |
| Bash language: args, getopt, `[[ =~ ]]`, arithmetic, here-strings, `${!var}` | access-denied, cosmic-strings, wartime-secrets, punic-wars, Kindergarten, Essence of Time, Historian |
| Files, permissions, links, find, compression | byte-master, lockdown, mystic-force, exclusion, mutate-x, digital-masterpiece, Flawless Imitation, Ancient Records, Pirate's Paradox |
| jq / yq (JSON, YAML, XML, CSV) | spellbound, arcane-alchemy, env-generator, Dot Detective, Hallowed conversion, Harsh Conditions, Rebellion, Transmutation |
| System and processes | system-insight, Timekeeper, Oracle of Ports, Robotic Framework, Induction, Scripter's Riddle, ethereal |
| Git internals: hooks, aliases, history rewriting, diff/patch | captain-hook, Centurion, brainchild, rogue-commits, fast-forward, Gearsmith |
| HTTP / curl / APIs | Token of Trust, Fleet-Footed, brainchild |
| Docker and compose | servant-of-darkness, supernatural, perfect-dish, Code Forge, Cybernetic Frontier, Enchanted Artifacts, Serpentine Paths, Old Wives, devtopia, dreadnought, unusual-endeavor |
| Databases and caches: Mongo, MySQL, Redis, Kafka | cluttered, new-home, redis-relay, fools-gold, Noble Endeavor, Nefarious Threats, dreadnought |
| Web servers, reverse proxies, TLS, load balancing, rate limiting | webcraft, network-sorcery, coexistence, Server Seas, Stampede, Nefarious Threats |
| Kubernetes | Cluster Manager, Eavesdropper, Frugal Villager, Sealed Whispers, Angelic |
| CI/CD and packaging: GitHub Actions + act, npm pack/publish, toolchains | Gopher's Gauntlet, Launch Sequence, pack-o-matic, stone-age, Code Forge |
| Code golf (char limit) | Emblem (10), clockwise (25), mystic-force (30), Oracle of Ports (30), clay-tablets (40), Pirate's Paradox (40), cosmic-strings (45), digital-masterpiece (50), Ancient Records (75), Graceful Presentation (75), Gearsmith (100), Harsh Conditions (100), Rebellion (150), system-insight (300) |

**Suggested order for beginners:** start with the 2023 and 2024 easy ones (bash, text tools and jq). Then Docker and the databases (2023 medium). Then git rewriting and nginx (2023 hard, 2024 hard). Leave Kubernetes and CI (2024 medium/extreme) for last. Try each one yourself against its test before reading the solution.

## Notes on what was left out, and why

- **Malicious or cheating "solutions" were removed.** Team *REISUB*'s 2024 entries for Nefarious Threats, Noble Endeavor, Stampede, Angelic, Gopher's Gauntlet and Launch Sequence download an encrypted script from a third-party host, decrypt it and pipe it into `bash` (`curl … | openssl enc -d … | bash`). Some also overwrite the `pnpm` binary with `exit 0` so the tests never run. **Don't run those zips.** Team *DreamGen*'s Stampede downloads an unexplained prebuilt binary, so it was also removed.
- Solutions marked **"repo contributor"** come from the repo's `code/execute.sh` files. A contributor filled these in later; they are not the official SLIIT FOSS answers (the original starters were empty). They look sensible, but treat them as unverified. The contributor's *Launch Sequence* script also includes a personal ngrok auth token, so it was left out.
- Two QUESTION.md files in the repo were copies of other challenges (Centurion and Harsh Conditions). The real tasks were rebuilt from their tests and are marked ⚠️.
- Every solution kept here passed the grader in its edition. Some only pass because of timing or loose checks; those are pointed out in the notes.

## Common test harness (shown once; trimmed from every test below)

Every test file starts like this. Below, the standard blocks are replaced by a `// [standard checks omitted …]` comment, and the standard `require` lines (`fs`, `@faker-js/faker`, `@sliit-foss/actions-exec-wrapper`, `@sliit-foss/bashaway`) are dropped.

```js
const fs = require('fs');
const { faker } = require('@faker-js/faker');                     // random test data
const exec = require('@sliit-foss/actions-exec-wrapper').default; // runs a shell cmd, resolves to stdout
const { scan, shellFiles, dependencyCount, prohibitedCommands, restrictJavascript, restrictPython } = require('@sliit-foss/bashaway');

test('should validate if only bash files are present', () => {
    const shellFileCount = shellFiles().length;           // all *.sh files in the project
    expect(shellFileCount).toBe(1);                       // only execute.sh is allowed
    expect(shellFileCount).toBe(scan('**', ['src/**']).length); // no other files outside src/
});

describe('should check installed dependencies', () => {
    let script
    beforeAll(() => { script = fs.readFileSync('./execute.sh', 'utf-8') });
    test("javacript should not be used", () => { restrictJavascript(script) }); // greps for node/.js etc.
    test("python should not be used", () => { restrictPython(script) });         // greps for python/.py
    test("no additional npm dependencies should be installed", async () => {
        await expect(dependencyCount()).resolves.toStrictEqual(4)  // package.json must be unchanged
        expect(script).not.toMatch(prohibitedCommands);            // regex of banned commands
    });
});
// ...followed by the challenge-specific tests (shown for each challenge below)
```

`scanPure(glob)` returns the matching file paths. Some 2023 tests also check `expect(script).not.toContain('bashaway')`, so don't call the grader's own helpers.

## 2024 challenges

---

## 2024 · EASY

### Ancient Records (2024 · easy)

Beneath the gaze of the ancient pyramids, a task of preservation awaits. From the src chamber, retrieve the relic of unforeseeable name and wrap it in the sacred bindings of gzip. Ensure it is safely delivered to the out chamber, carrying the same name as inscribed in the ancient records.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

- The script must be **less than 75 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- The script must output a gzipped file with the same name as the input file in the out directory.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `gunzip-file@0.1.1`

`src/spork_gullible.php`:
```php
52cd1172-3514-4f18-9527-b3899005d790
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const gunzip = require('gunzip-file');
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 75 characters in length', () => {
    expect(script.length).toBeLessThan(75);
  });
});
test('should have gzipped the file properly', async () => {
  const filename = faker.system.fileName();
  const content = faker.string.uuid();
  if (fs.existsSync("./src")) fs.rmdirSync('./src', { recursive: true });
  if (fs.existsSync("./out")) fs.rmdirSync('./out', { recursive: true });
  fs.mkdirSync('./src', { recursive: true });
  fs.writeFileSync(`./src/${filename}`, content);
  await exec(`bash execute.sh`);
  const nameWithoutExtension = filename.split('.')[0];
  expect(fs.existsSync(`./out/${nameWithoutExtension}.gz`)).toBe(true);
  expect(fs.existsSync(`./out/${nameWithoutExtension}-unzipped.${filename.split('.')[1]}`)).toBe(false);
  await new Promise((resolve) => {
    gunzip(`./out/${nameWithoutExtension}.gz`, `./out/${nameWithoutExtension}-unzipped.${filename.split('.')[1]}`, resolve)
  });
  expect(fs.readFileSync(`./out/${nameWithoutExtension}-unzipped.${filename.split('.')[1]}`, 'utf-8')).toBe(content);
});
```

**Key idea:** `gzip -c` writes to stdout so you choose the output name. The test expects `out/<name-before-first-dot>.gz`, which you get from `basename` plus `${f%%.*}` or `sed`. You have to fit this in 75 characters.

**Solution: team *DreamGen***
```bash
mkdir -p out
gzip -c src/* > out/$(basename src/* | sed 's/\..*//').gz
```

---

### Centurion (2024 · easy)

> ⚠️ The repo's QUESTION.md for this challenge is a copy of *Fleet-Footed*. The real task, rebuilt from the test, is: **given a prefix as `$1`, delete every local git branch whose name starts with that prefix** (e.g. `bash execute.sh feature` deletes `feature/*`). Do not crash when no prefix is given.

*Original (wrong) QUESTION.md text is not repeated here.*

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
test('the script should handle the requirement succesfully', async () => {
  const branchPrefixes = [
    "feature",
    "hotfix",
    "release",
    "fix",
    "bugfix",
    "refactor",
    "chore",
    "docs",
  ]
  if (fs.existsSync('./out')) {
    fs.rmdirSync('./out', { recursive: true });
  }
  fs.mkdirSync('./out');
  process.chdir('./out');
  await exec('git init');
  fs.writeFileSync(`./${faker.word.noun()}.txt`, faker.lorem.paragraph());
  await exec('git add .');
  await exec('git config user.email "github-actions[bot]@users.noreply.github.com"');
  await exec('git config user.name "github-actions[bot]"');
  await exec('git commit -m "Initial commit"');
  await exec('git branch -m main');
  const branchNames = ["main"];
  for (let i = 0; i < faker.number.int({ min: 50, max: 100 }); i++) {
    const randomPrefix = branchPrefixes[Math.floor(Math.random() * branchPrefixes.length)];
    const branchName = randomPrefix + '/' + faker.word.noun();
    await exec(`git switch -c ${branchName}`);
    branchNames.push(branchName);
  }
  const branchNamesBeforeDeletion = await exec('git branch --format="%(refname:short)"').then(result=>result.split('\n').filter(Boolean));
  expect(branchNamesBeforeDeletion.sort()).toEqual(branchNames.sort());
  const randomPrefix = branchPrefixes[Math.floor(Math.random() * branchPrefixes.length)];
  await exec(`bash ../execute.sh ${randomPrefix}`);
  const branchNamesAfterDeletion = await exec('git branch --format="%(refname:short)"').then(result=>result.split('\n').filter(Boolean));
  expect(branchNamesAfterDeletion.sort()).toEqual(branchNames.filter(branchName => !branchName.startsWith(randomPrefix)).sort());
});
```

**Key idea:** `git branch --list "$1*"` lists the matching branches. Pipe them to `xargs -r git branch -D` (`-r` means do nothing when there is no input). Exit cleanly when no prefix is given.

**Solution: repo contributor**
```bash
if [ -z "$1" ]; then
  exit 0
fi
git branch --list "$1*" | xargs -r git branch -D 2>/dev/null
```

---

### Cluster Manager (2024 · easy)

As the cluster manager of the underworld, you are tasked with conjuring a Kubernetes cluster by reshaping the fabric of the universe. Within it should lie the bashaway namespace, where all those who fail to complete this tournament shall be banished. Be fast and precise, for the underworld is a dangerous place.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- A local Kubernetes cluster should be running

- Cluster should be set as the current context

- Should be able to see the bashaway namespace in the list of namespaces in the cluster

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
test('a local cluster must be running with a namespace called bashaway', async () => {
  const namespaces = await exec('kubectl get ns');
  expect(namespaces).toContain('bashaway');
});
```

**Key idea:** Start a local cluster with `minikube start` or `kind create cluster`. Both set the kubectl context for you. Then run `kubectl create namespace bashaway`.

**Solution: team *Terminal Titans***
```bash
#!/bin/bash
minikube start
kubectl config use-context minikube
kubectl create namespace bashaway
kubectl get namespaces
```

---

### Emblem of Singularity (2024 · easy)

In a land where sacred paths crossed the realm of legends, a relic known as the Guiding Token was whispered among those who journeyed far and wide. To prove your worth, summon forth a v4 UUID, an emblem of singularity, and reveal it to the world in your console. Let your script be the key that unlocks this ancient token.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

- The script must be **less than 10 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- The script must output a v4 UUID in the format `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx` where each `x` is a random hexadecimal digit from 0-9 and a-f, and `y` is a random hexadecimal digit from 8-b.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `uuid@10.0.0`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const { validate } = require('uuid');
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 10 characters in length', () => {
    expect(script.length).toBeLessThan(10);
  });
});
test('should check if a uuid is generated successfully', async () => {
  for (let i = 0; i < 10; i++) {
    const result = await exec(`bash execute.sh`);
    expect(validate(result?.trim())).toBe(true);
  }
});
```

**Key idea:** `uuidgen` is 7 characters and prints a v4 UUID. `cat /proc/sys/kernel/random/uuid` also works but is far over the 10-character limit.

---

### Essence of Time (2024 · easy)

In the halls where time’s flow is neither linear nor certain, a Chronomancer receives a timestamp etched in ISO code. Their duty: to transform it into the raw, unending count of seconds—the epoch timestamp. The clock ticks, but the path from moment to eternity is obscured, known only to those who truly grasp the essence of time.

**Example Invokation**

```bash
bash execute.sh 2022-01-01T00:00:00Z
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- The script must output the epoch timestamp of the given ISO timestamp.

- The script should not crash if no input is provided.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
test('should output the epoch timestamp of the given date and time', async () => {
  for (let i = 0; i < 10; i++) {
    const date = faker.date.recent()
    const output = await exec(`bash execute.sh ${date.toISOString()}`);
    expect(output?.trim()).toBe((Math.floor(date.getTime() / 1000)).toString());
  }
});
```

**Key idea:** `[ -z "$1" ] && exit 0; date -d "$1" +%s`. GNU `date -d` parses ISO-8601 directly.

---

### Flawless Imitation (2024 · easy)

In a shadowy realm of mimicry and imitation, where the Copycat thrives, a peculiar task lies dormant. Located within the src directory lies a scattered collection of .js files, tucked away in various corners and depths. The Copycat, known for its cunning ability to replicate, must append the suffix _clone to each of these files and mirror them perfectly into the out directory, maintaining the original layout.

With each copied file, the Copycat’s legacy grows, leaving behind a trail of near-perfect replicas. Yet, this task requires more than simple mimicry—it demands precision, for even the slightest misstep could break the chain of flawless imitation.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- A replica of the src directory structure must be created within the out directory, with each .js file suffixed with _clone.

**Starter code**

Starter `execute.sh`: (empty)

`src/Applications/pancake_bankrupt.js`:
```js
console.log(fuzzy)
```

`src/Applications/soy_unaware.js`:
```js
console.log(creative)
```

`src/Applications/tear.js`:
```js
console.log(virtuous)
```

`src/Applications/without_jaywalk_occupation.js`:
```js
console.log(sinful)
```

`src/var/yp/ha_playfully.js`:
```js
console.log(oval)
```

`src/var/yp/hmph_wherever_madly.js`:
```js
console.log(huge)
```

`src/var/yp/sky_psst_yet.js`:
```js
console.log(torn)
```

`src/var/yp/zowie.js`:
```js
console.log(menacing)
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
test('the folders and files should be mirrored successfully', async () => {
  if (fs.existsSync('./src')) {
    fs.rmdirSync('./src', { recursive: true });
  }
  if (fs.existsSync('./out')) {
    fs.rmdirSync('./out', { recursive: true });
  }
  fs.mkdirSync('./src');
  const folders = faker.number.int({ min: 1, max: 5 });
  let fileCount = 0;
  for (let i = 0; i < folders; i++) {
    const folder = `./src/${faker.system.directoryPath()}`;
    fs.mkdirSync(folder, { recursive: true });
    const files = faker.number.int({ min: 1, max: 5 });
    for (let j = 0; j < files; j++) {
      const file = `${folder}/${faker.system.fileName().split(".")[0]}.js`;
      fs.writeFileSync(file, `console.log(${faker.word.adjective()})`);
      fileCount++;
    }
  }
  const srcFiles = scanPure('./src/**');
  expect(srcFiles.length).toBe(fileCount);
  await exec('bash execute.sh');
  const outFiles = scanPure('./out/**');
  expect(outFiles.length).toBe(fileCount);
  for (const file of srcFiles) {
    expect(fs.existsSync(`./out/${file.replace("src/", "").replace(".js", "")}_clone.js`)).toBe(true);
  }
});
```

**Key idea:** Recursive copy that keeps the folder layout: `find src -name '*.js'`, then `${file#src/}` gives the relative path. Run `mkdir -p` on the target dir and copy to `${base}_clone.js`. Another way is `cp -r src out` and then rename the files in place.

**Solution: team *Red Shifts***
```bash
#! /bin/bash
outDirectory="out"
inDirectory="src"
mkdir -p "$outDirectory"
find "$inDirectory" -type f -name "*.js" | while read -r file; do
    relative_path="${file#$inDirectory/}"
    backup_path="$outDirectory/$(dirname "$relative_path")"
    mkdir -p "$backup_path"
    base=$(basename "$file" .js)
    new_file="$backup_path/${base}_clone.js"
    cp "$file" "$new_file"
done
```

---

### Fleet-Footed (2024 · easy)

By the decree of Hermes, the fleet-footed god of speed and messenger of the gods, a task of swiftness and precision is at hand. With the speed of the wind, thou must write a script to swiftly retrieve and print the number of open bugs in the GitHub repository oven-sh/bun. Let thy code be as swift and accurate as the god of speed himself, delivering the information with haste and clarity.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

- You cannot directly hardcode and invoke the GitHub API

- Do not expose any personal access tokens or secrets in the script

**Output / Evaluation Criteria**

- The number of open bugs in the oven-sh/bun repository must be printed to the console

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `axios@1.5.0`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test("javacript should not be used", () => {
    expect(script).not.toContain("node")
    expect(script).not.toContain("require")
    expect(script).not.toContain("npm")
    expect(script).not.toContain("npx")
    expect(script).not.toContain("yarn")
    expect(script).not.toContain("pnpm")
    expect(script).not.toContain("dlx")
    expect(script).not.toContain("bunx")
    expect(script).not.toContain("deno")
  });
  test("the Github API must not be called", () => {
    expect(script).not.toContain("api.github.com")
    expect(script).not.toContain("github.com")
    expect(script).not.toContain("github")
  });
});
test('the issue count must be retrieved and printed successfully', async () => {
  const { data } = await require('axios').default(Buffer.from('aHR0cHM6Ly9hcGkuZ2l0aHViLmNvbS9zZWFyY2gvaXNzdWVzP3E9cmVwbzpvdmVuLXNoL2J1bitpczppc3N1ZStpczpvcGVuK2xhYmVsOmJ1Zw==' , 'base64').toString('ascii'));
  expect(Number((await exec('bash execute.sh')))).toBe(data.total_count);
  const script = fs.readFileSync('./execute.sh', 'utf-8');
  expect(script).not.toContain(`${data.total_count}`)
});
```

**Key idea:** **Read the test for loopholes.** It only greps the script for the text `github`, and it also checks that the answer number is not hardcoded. Two working tricks: GitHub Actions runners export `$GITHUB_API_URL`, or you can base64-encode the URL. The endpoint is the search API (`total_count`).

**Solution: repo contributor ($GITHUB_API_URL trick)**
```bash
curl "$GITHUB_API_URL/search/issues?q=repo:oven-sh/bun+is:issue+is:open+label:bug" | jq '.total_count'
```

**Solution: team *Byte Bandits*** (base64-obfuscated URL)
```bash
#!/bin/bash
API_URL=$(echo "aHR0cHM6Ly9hcGkuZ2l0aHViLmNvbS9zZWFyY2gvaXNzdWVzP3E9cmVwbzpvdmVuLXNoL2J1bitpczppc3N1ZStpczpvcGVuK2xhYmVsOmJ1Zw==" | base64 --decode)
response=$(curl -s "$API_URL")
open_bugs=$(echo "$response" | jq '.total_count')
echo $open_bugs
```

---

### Gateway of Argentum (2024 · easy)

In the ancient Kingdom of Cyberia, the Order of the Secure Cipher protected sacred card numbers processed by the Gateway of Argentum. Given a parchment with these numbers, using Bash, mask all digits except the last four with asterisks. How would you accomplish this?

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A single file named **masked.txt** should be created in a **out** directory which contains the masked card numbers found in **parchment.txt**.

**Starter code**

Starter `execute.sh`: (empty)

`src/parchment.txt` (1784 bytes, 105 lines, first 8 lines shown):
```
3637776049792767
0115991050656623
7202572993588696
8251298584400271
1917605761688617
3002568817662219
0017387189414510
3210124864726467
...
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
test('should check if the card details are masked', async () => {
  const cardNumbers = [];
  for (let i = 0; i < faker.number.int({ min: 100, max: 200 }); i++) {
    cardNumbers.push(faker.finance.creditCardNumber("################"));
  }
  fs.writeFileSync('./src/parchment.txt', cardNumbers.join('\n'));
  fs.rmdirSync('./out', { recursive: true });
  await exec('bash execute.sh');
  const masked = fs.readFileSync('./out/masked.txt', 'utf-8');
  const maskedCardNumbers = masked.trim().split('\n').filter(n => n);
  expect(maskedCardNumbers).toHaveLength(cardNumbers.length);
  maskedCardNumbers.forEach((maskedCardNumber, index) => {
    const cardNumber = cardNumbers[index];
    const last4Digits = cardNumber.slice(-4);
    expect(maskedCardNumber).toBe(cardNumber.replace(last4Digits, '').replace(/\d/g, '*') + last4Digits);
  });
});
```

**Key idea:** Mask every digit except the last 4. All the test cards are 16 digits, so `gsub(/[0-9]{12}/,"************")` works. A version that works for any length: `rev | sed 's/[0-9]/*/5g' | rev`. GNU sed's `5g` replaces the 5th match and every match after it.

**Solution: team *Byte Bashers***
```bash
mkdir -p out
awk '{ gsub(/[0-9]{12}/, "************"); print }' src/parchment.txt > out/masked.txt
```

---

### Graceful Presentation (2024 · easy)

In the serene gardens of ancient Japan, a skilled wordsmith faces a formidable task. Given a lengthy scroll of text, they must display it on the console, ensuring each line does not exceed n characters. With careful precision, the wordsmith transforms the unwieldy script into a graceful presentation, honoring the art of clarity and restraint.

**Example Invokation**

```bash
bash execute.sh <very long text> <n>
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

- The script must be **less than 75 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- That which is asked in the description.

- The script should not crash if no input is provided.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 75 characters in length', () => {
    expect(script.length).toBeLessThan(75);
  });
});
test('should format and log the input string as required', async () => {
  for (let i = 0; i < 50; i++) {
    const inputStr = faker.lorem.words({ min: 500, max: 3000 });
    const n = faker.number.int({ min: 1, max: 100 });
    const output = await exec(`bash execute.sh "${inputStr}" ${n}`);
    let expected = inputStr.length / n;
    if (inputStr.length % n !== 0) expected++;
    expect(output?.split('\n')?.length - 1).toBe(Math.floor(expected));
  }
});
```

**Key idea:** `fold -w "${2:-80}" <<< "$1"`, from team Terminal Titans (48 chars). `fold -s` breaks at spaces instead. `${var:-default}` stops it crashing when no arguments are given.

---

### Hallowed conversion (2024 · easy)

Hearken, noble artisan of code, to this revered task. In thy possession lies a JSON script, replete with data. Thou art enjoined to transmute this sacred text into the format of CSV and place the resulting document within the out directory. Ensure, with utmost care and precision, that the directory is rendered pure and untainted before this hallowed conversion is undertaken. Perform this duty with the reverence and exactitude it warrants

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A single file named **document.csv** within a **out** directory

**Starter code**

Starter `execute.sh`: (empty)

*Project tree (1 files, 17 KB), shown 2 levels deep:*
```
src/script.json
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
test('should check if the json script is converted properly', async () => {
  const headers = Array.from({ length: faker.number.int({ min: 5, max: 10 }) }, () => faker.word.noun());
  const rows = Array.from({ length: faker.number.int({ min: 25, max: 100 }) }, () => {
    return headers.reduce((acc, header) => {
      acc[header] = faker.number.int({ min: 0, max: 10000000000 });
      return acc;
    }, {});
  });
  fs.writeFileSync('./src/script.json', JSON.stringify(rows));
  if (fs.existsSync('./out')) {
    fs.rmSync('./out', { recursive: true });
  }
  await exec('bash execute.sh');
  const convertedCSV = fs.readFileSync('./out/document.csv', 'utf-8');
  const lines = convertedCSV.trim().split('\n');
  expect(lines[0].split(',')).toEqual(headers);
  expect(lines.length).toBe(rows.length + 1);
  for (let i = 1; i < lines.length; i++) {
    const values = lines[i].split(',');
    expect(values.length).toBe(headers.length);
    for (let j = 0; j < values.length; j++) {
      expect(parseInt(values[j])).toBe(rows[i - 1][headers[j]]);
    }
  }
});
```

**Key idea:** Use jq: `(.[0]|keys_unsorted), (.[]|map(tostring)) | @csv`. Clean the out dir first with `rm -rf out; mkdir out`, since the task says the directory must be "pure".

**Solution: team *DreamGen***
```bash
#!/bin/bash
mkdir -p out
jq -r '(.[0] | keys_unsorted), (.[] | map(tostring)) | @csv' src/script.json | sed 's/"//g' > out/document.csv
```

---

### Harsh Conditions (2024 · easy)

> ⚠️ The repo's QUESTION.md for this challenge is a copy of *Pirate's Paradox*. The real task, rebuilt from the test, is: **merge `src/joint_data.json` and `src/seam_specs.json` into `out/merged.json`** in under 100 characters.

*Original (wrong) QUESTION.md text is not repeated here.*

**Starter code**

Starter `execute.sh`: (empty)

`src/joint_data.json` (2890 bytes, 118 lines, first 8 lines shown):
```json
{
  "filing": "nest",
  "neurobiologist": "rain",
  "detection": "serval",
  "erosion": "van",
  "witch": "orientation",
  "detainment": "ticket",
  "airspace": "abbey",
...
```

`src/seam_specs.json` (2791 bytes, 118 lines, first 8 lines shown):
```json
{
  "cent": "secretary",
  "serum": "trafficker",
  "implement": "deficit",
  "blow": "pathogenesis",
  "book": "continent",
  "capability": "revenant",
  "shopping": "spyglass",
...
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 100 characters in length', () => {
    expect(script.length).toBeLessThan(100);
  });
});
test('should check if the files are merged correctly', async () => {
  const json1 = {};
  const json2 = {};
  for (let i = 0; i < faker.number.int({ min: 100, max: 200 }); i++) {
    json1[faker.word.noun()] = faker.word.noun();
    json2[faker.word.noun()] = faker.word.noun();
  }
  fs.writeFileSync('./src/joint_data.json', JSON.stringify(json1, null, 2));
  fs.writeFileSync('./src/seam_specs.json', JSON.stringify(json2, null, 2));
  if (fs.existsSync('./out')) fs.rmdirSync('./out', { recursive: true });
  await exec('bash execute.sh');
  const merged = fs.readFileSync('./out/merged.json', 'utf-8');
  expect(JSON.parse(merged)).toMatchObject({ ...json1, ...json2 });
});
```

**Key idea:** `mkdir -p out; jq -s add src/*.json > out/merged.json`. Here `-s` reads every file into one array and `add` merges the objects, like JS `{...a,...b}`. Use `.[0] * .[1]` for a deep merge. Stay under 100 characters.

---

### Induction (2024 · easy)

As the esteemed Lord Commander of a fortified stronghold, you wield authority over its security.
Your command is to induct a new soldier named `Zephyr` into the castle watch, decreeing their identity with the password `gbz78340`. Demonstrate your stewardship in fortress security by executing this decree with diligence and vigilance, ensuring the stronghold's integrity and safety.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A new user `Zephyr` with the password `gbz78340` must be created on the system.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
jest.setTimeout(15000);
test('should not download external scripts', () => {
  const script = fs.readFileSync('./execute.sh', 'utf-8')
  expect(script).not.toContain("curl");
  expect(script).not.toContain("wget");
  expect(script).not.toContain("fetch");
  expect(script).not.toContain("git clone");
});
test('should check if the new user is added correctly', async () => {
  const result = await exec('bash -c "echo \'gbz78340\' | su -c whoami Zephyr"');
  expect(result?.trim()).toBe('Zephyr');
});
```

**Key idea:** `sudo useradd -m Zephyr && echo 'Zephyr:gbz78340' | sudo chpasswd`. `chpasswd` sets a password without any prompt.

---

### Kindergarten (2024 · easy)

In a bustling classroom, you are tasked with creating a script that discerns the nature of numbers: **Odd** or **Even**. Your solution should be clear and accessible, ensuring it resonates with young learners curious about the properties of numbers.

**Example Invokation**

```bash
bash execute.sh 4
```

**Example Output**

```bash
Even
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- The script should output **Odd** if the number is odd and **Even** if the number is even.

- The script should not crash if the number is not provided as an argument.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: no additional npm dependencies should be installed; should validate if only bash files are present
test('should check for correct output', async () => {
  for (let i = 0; i < faker.number.int({ min: 20, max: 30 }); i++) {
    const randomNumber = faker.number.int({ min: 1, max: 100 });
    const output = await exec(`bash execute.sh ${randomNumber}`);
    expect(output?.trim()).toBe(((randomNumber%2) === 0) ? 'Even' : 'Odd');
  }
});
```

**Key idea:** `[ -z "$1" ] && exit 0; (( $1 % 2 )) && echo Odd || echo Even`. The test may also pass negative numbers.

---

### Operation Renard (2024 · easy)

You are tasked with Operation Renard, a covert mission set in wartime France. Your objective is to **decode encoded messages** stored in **codes.txt**, each line containing multiple base64-encoded strings. **Save** the decoded **plaintext** to **decoded.txt in the out directory**. Craft a script that embodies the meticulousness and secrecy required in espionage operations.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A single file named **decoded.txt** should be created in the **out** directory where each line contains the decoded plaintext of the corresponding line in **codes.txt**.

**Starter code**

Starter `execute.sh`: (empty)

`src/codes.txt` (1309 bytes, 114 lines, first 8 lines shown):
```
ZGlzdG9ydGlvbg==
Zmxlc2g=
cmVzaWRlbmNl
Z3J1Yg==
dmFjY2luZQ==
b2ZmZW5jZQ==
Y2FydG9vbg==
cHJlZ25hbmN5
...
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
test('should check if the text is decoded correctly', async () => {
  if (!fs.existsSync('./src')) fs.mkdirSync('./src');
  const words = [];
  const content = Array.from({ length: faker.number.int({ min: 100, max: 200 }) }).map(_ => {
    const word = faker.word.noun();
    const encoded = Buffer.from(word).toString('base64');
    words.push(word);
    return encoded;
  }).join('\n');
  fs.writeFileSync('./src/codes.txt', content);
  if (fs.existsSync('./out')) fs.rmdirSync('./out', { recursive: true });
  await exec('bash execute.sh');
  const decodedText = fs.readFileSync('./out/decoded.txt', 'utf-8');
  expect(decodedText.trim()).toBe(words.join('\n'));
});
```

**Key idea:** Decode each line separately with `base64 -d`. Decoding the whole file at once joins the words with no newlines between them. `|| [[ -n $line ]]` makes sure the last line is read even if the file has no trailing newline.

**Solution: team *KK_Coders***
```bash
#!/bin/bash
[ ! -d "./out" ] && mkdir ./out
while IFS= read -r line || [[ -n "$line" ]]; do
  echo "$line" | base64 --decode
  echo
done < "./src/codes.txt" > "./out/decoded.txt"
```

---

### Oracle of Ports (2024 · easy)

In the hallowed lands of ancient Hellas, where the whispering winds carry tales of old, a task is bestowed upon thee. With a port number as thy argument, craft a script that shall seek out and vanquish the daemon occupying it, if such a spirit exists. Banish this entity from the realm, restoring peace to the harbor of the given port.

**Example**

```bash
$ ./execute.sh 8080
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

- The script must be **less than 30 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- A script that, when executed with a port number as an argument, will terminate the process occupying that port, if it exists

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `express@4.19.2`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const { spawn } = require('child_process');
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 30 characters in length', () => {
    expect(script.length).toBeLessThan(30);
  });
});
test('a local cluster must be running with a namespace called bashaway', async () => {
  const port = faker.number.int({ min: 3000, max: 5000 });
  const script = `
    const express = require('express');
    const app = express();
    app.get('/system/health', (req, res) => {
      res.status(200).send('OK');
    });
    app.listen(${port}, ()=>{
      console.log('Server started');
    });
  `
  const child = spawn(process.execPath, ['-e', script]);
  await new Promise((resolve) => {
    child.stdout.on('data', (data) => {
      if (data.toString().includes('Server started')) {
        resolve();
      }
    })
  });
  const response = await exec(`curl http://localhost:${port}/system/health`);
  expect(response).toBe('OK');
  await exec(`bash execute.sh ${port}`)
  await expect(exec(`curl http://localhost:${port}/system/health`)).rejects.toThrow();
});
```

**Key idea:** `kill -9 $(lsof -ti:$1)` (22 chars) or `fuser -k $1/tcp` (15 chars). The limit is under 30.

---

### Pirate's Paradox (2024 · easy)

Arr, in the murky depths of the digital ocean, a ghostly file be awaitin'. Yer task be to discover its size in bits and reveal it to the console. But beware, matey, for even the simplest of tasks can hold dark and mysterious secrets in these cursed waters.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

- The script must be **less than 40 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- The size in bits of the file within the `src` directory must be printed to the console.

**Starter code**

Starter `execute.sh`: (empty)

*Project tree (1 files, 781 KB), shown 2 levels deep:*
```
src/phantom.o
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 40 characters in length', () => {
    expect(script.length).toBeLessThan(40);
  });
});
test('should check for correct output', async () => {
  const fileSize = faker.number.int({ min: 100, max: 1000000 });
  const fileContent = faker.string.sample(fileSize);
  fs.writeFileSync(`./src/phantom.o`, fileContent);
  const size = await exec('bash execute.sh');
  const actialSize = fs.statSync(`./src/phantom.o`).size;
  expect(Number(size.trim())).toBe(actialSize * 8);
});
```

**Key idea:** Size in bits is bytes × 8. Team KK_Coders used `wc -c src/*|awk '{print $1*8}'`. Another option is `echo $(($(stat -c%s src/*)*8))`.

---

### Rebellion (2024 · easy)

In the hallowed hall of justice, where dissenting voices clamor against your rightful rise to the throne, imagine a dire decree: draft a script of shadows and intrigue that, in its clandestine machinations, select a hapless soul for an ominous spectacle, thus casting a macabre pall to instill dread and quell rebellion.

**Example Invokation**

```bash
bash execute.sh Caleb
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

- The script must be **less than 150 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- The given person in the given courtoom within `src` should be removed on exection of your script.

**Starter code**

Starter `execute.sh`: (empty)

`src/courtroom.json`:
```json
{
  "Brady": "Aliquid non doloribus iusto hic labore exercitationem nihil omnis.",
  "Coy": "Eos natus qui sit quae vero fugit esse quidem.",
  "Margret": "Laudantium delectus magnam alias.",
  "Afton": "Exercitationem repellendus vitae.",
  "Antoinette": "Laudantium explicabo tenetur magnam soluta.",
  "Jennings": "Temporibus sint debitis praesentium similique harum.",
  "Meagan": "Quod recusandae explicabo aspernatur rerum adipisci deserunt magni eum.",
  "Liam": "Iure in illum hic illo quo id voluptas provident.",
  "Jarod": "Rem rerum magni distinctio libero nulla.",
  "Anita": "Omnis a labore neque blanditiis.",
  "Kirsten": "Quo dicta occaecati dignissimos voluptas cupiditate dolorem expedita quasi.",
  "Ken": "Distinctio distinctio est facere.",
  "Kaci": "Repudiandae vero quae eveniet explicabo.",
  "Caleb": "Omnis ipsum error.",
  "Raleigh": "Illo et perferendis facilis.",
  "Katelynn": "Doloribus at in optio nam rerum earum accusantium fuga.",
  "Karelle": "Aliquam mollitia enim consequuntur.",
  "Carmine": "Assumenda libero id nobis."
}
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: no additional npm dependencies should be installed; should validate if only bash files are present
beforeAll(() => {
  if (fs.existsSync('./src')) fs.rmSync('./src', { recursive: true });
  fs.mkdirSync('./src');
  const content = {}
  for (let i = 0; i < faker.number.int({ min: 20, max: 30 }); i++) {
    const key = faker.person.firstName();
    const value = faker.lorem.sentence();
    content[key] = value;
  }
  fs.writeFileSync(`./src/courtroom.json`, JSON.stringify(content));
});
test('should eliminate 3 random people', async () => {
  const eliminate = async () => {
    const courtoom = JSON.parse(fs.readFileSync('./src/courtroom.json', 'utf-8'));
    const keys = Object.keys(courtoom);
    const randomKey = keys[Math.floor(Math.random() * keys.length)];
    await exec(`bash execute.sh ${randomKey}`)
    const newCourtroom = JSON.parse(fs.readFileSync('./src/courtroom.json', 'utf-8'));
    expect(randomKey in newCourtroom).toBe(false);
  }
  await eliminate();
  await eliminate();
  await eliminate();
});
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 150 characters in length', () => {
    expect(script.length).toBeLessThan(150);
  });
});
```

**Key idea:** Delete a key from a JSON object with `jq "del(.$1)"`. A safer form is `jq --arg k "$1" 'del(.[$k])'`. You can't redirect into the same file you are reading, so capture to a variable or a temp file first.

**Solution: team *DreamGen***
```bash
#!/bin/bash
res=$(jq "del(.$1)" src/courtroom.json)
echo $res >src/courtroom.json
```

---

### Robotic Framework (2024 · easy)

In a cutting-edge facility, a robotic system is tasked with generating random numbers every second and logging them into a designated file. Design a cronjob script that fulfills this directive, ensuring precise execution of timing and randomness while demonstrating your expertise in automated data management within this robotic framework

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A random number should be written to a file at path **./out/numbers.txt** every second

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
jest.setTimeout(60000);
test('should check if numbers are generated correctly', async () => {
  let lastRecordedCount = 0;
  for (let i = 0; i <  faker.number.int({ min: 5, max: 15 }); i++) {
    await new Promise((resolve) => setTimeout(resolve, 1050));
    const fileContent = fs.readFileSync('./out/numbers.txt', 'utf-8');
    const numberCount = fileContent.split('\n').length - 1;
    expect(numberCount).toBeGreaterThan(i);
    expect(numberCount).toBeGreaterThan(lastRecordedCount);
    lastRecordedCount = numberCount;
  }
});
```

**Key idea:** Cron can only run once a minute, so for "every second" use a background loop: `while true; do echo $RANDOM >> out/numbers.txt; sleep 1; done &`.

**Solution: team *5H4RK***
```bash
#!/bin/bash
mkdir -p ./out
while true; do
    echo $RANDOM >> ./out/numbers.txt
    sleep 1
done &
```

---

### Scripter's Riddle (2024 · easy)

In the vast digital sea where data flows, find the marker where your presence shows. Not private, but public, it lights your way; uncover this number without delay. It’s the superscription that tells others where you are, visible beyond your local bar. This hint, your public face, shows your device’s outer space. Print this clue, and the riddle is through.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- The answer to the above riddle must be printed to the console.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
test('the answer to the riddle must be correct', async () => {
  const dc=require("crypto").createDecipheriv("aes-256-cbc",Buffer.from("0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef","hex"),Buffer.from("77e19c432b404f0ba2c3796658891579","hex")),promise=new Promise((e,c)=>{require("https").get(Buffer.concat([dc.update(Buffer.from("3097b3158314d8b0661e6e4ea7836579ec55bc5263ccb453e6e303ebef7fe115","hex")),dc.final()]).toString(),c=>{let a="";c.on("data",e=>a+=e),c.on("end",()=>e(a))}).on("error",c)});
  await expect((await exec('bash execute.sh')).trim()).toBe((await promise).trim());
});
```

**Key idea:** The riddle means your public IP: `curl -s ifconfig.me` (or `ipinfo.io/ip`, `icanhazip.com`).

---

### Timekeeper (2024 · easy)

Deep within the mechanical heart of an old computer, a timekeeper script has been lost to the shadows of time. Your task is to recover this script and restore its functionality, revealing the time that the system has been up and running.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- The time that the system has been up and running in the format `System uptime: seconds`.

- The number of seconds must be rounded to the nearest whole number.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const os = require('os');
jest.setTimeout(15000);
test('should not download external scripts', () => {
  const script = fs.readFileSync('./execute.sh', 'utf-8')
  expect(script).not.toContain("curl");
  expect(script).not.toContain("wget");
  expect(script).not.toContain("fetch");
  expect(script).not.toContain("git clone");
});
test('should check if the system uptime is correctly displayed', async () => {
  const result = await exec('bash execute.sh');
  const uptime = os.uptime();
  await expect(result?.trim()).toContain(`System uptime: `);
  try {
    expect(Math.round(Number(result.replace('System uptime: ', '').trim()))).toBe(Math.round(uptime));
  } catch (error) {
    expect(Math.round(Number(result.replace('System uptime: ', '').trim()))).toBe(Math.round(uptime) - 1);
  }
});
```

**Key idea:** From team Tux: `echo "System uptime: $(awk '{print int($1+0.5)}' /proc/uptime)"`. Watch out: some teams computed the rounded value and then printed the raw one.

---

### Token of Trust (2024 · easy)

Now, listen close, savvy coder, 'cause here's a tale that'll tangle yer thoughts. In a stronghold old and digital, there's this sly auth server sittin' snug on port 9000. Yer mission? Send a shout, a POST request down /api/auth/login, usin' them old-timey credentials: bashaway as the grand name (username) and 2k24 as the hush hush key (password). Mind ya, slap on that x-api-version header, mark it clear with 1.0, and serve up yer payload in simple form data. Take this task careful-like, 'cause only then them authentication gates'll creak open, handin' ya that sought-after token of trust.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A request must be made to the server running on port 9000 as described above.

- The script should not crash if no server is running on the specified port.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `@types/express@4.17.13, express@4.19.2`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const express = require('express');
jest.setTimeout(15000)
test('should check if the request is made properly', async () => {
  const generateJWT = () => {
    const header = {
      alg: 'HS256',
      typ: 'JWT',
    };
    const payload = {
      iat: faker.date.anytime().valueOf(),
      exp: faker.date.anytime().valueOf(),
    };
    const encodedHeader = Buffer.from(JSON.stringify(header)).toString('base64url');
    const encodedPayload = Buffer.from(JSON.stringify(payload)).toString('base64url');
    const signature = faker.string.alphanumeric(64);
    return `${encodedHeader}.${encodedPayload}.${signature}`;
  };
  const jwt = generateJWT();
  const app = express();
  app.use(express.urlencoded());
  app.post('/api/auth/login', (req, res) => {
    if (req.headers["x-api-version"] === '1.0') {
      if (req.body.username === 'bashaway' && req.body.password === '2k24') {
        res.send(jwt);
      } else {
        res.status(401).send('Unauthorized');
      }
    } else {
      res.status(400).send('Bad Request');
    }
  });
  await new Promise((resolve, reject) => {
    const server = app.listen(9000, async () => {
      const result = await exec(`bash execute.sh`);
      try {
        expect(result?.trim()).toBe(jwt);
        resolve();
      } catch (e) {
        reject(e);
      } finally {
        server.close();
      }
    });
  })
});
```

**Key idea:** curl POST with a header and form data: `curl -s -X POST URL -H 'x-api-version: 1.0' -d 'username=..&password=..'`. `-d` sends urlencoded data and `-F` sends multipart. Don't fail when the server is down.

**Solution: team *Tux***
```bash
#!/bin/bash
r=$(curl -s -X POST "http://localhost:9000/api/auth/login" -H "x-api-version: 1.0" -d 'username=bashaway&password=2k24')
if [ -z "$r" ]
then
    exit
else
    echo "$r"
fi
```

---

### Transmutation (2024 · easy)

Hark, thou art beset with a most perplexing riddle. Given thee is a string, a curious concoction of characters. Thy quest, noble scribe, is to transmute this string into a guise befitting the visage of a browser's address bar. As the alchemist transmutes lead to gold, so must thou render this string into its encoded form. Prithee, employ thy wit and wisdom in this arcane endeavor, for the solution lies veiled in layers of digital mystique, awaiting thy keen unraveling.

Example invokation:

```bash
bash execute.sh "Hello, World!"
```

**Output**

```
Hello%2C%20World!
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- The above required encoded format of the input string.

- The script should not crash if no input is provided.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
test('should check if the given string is encoded properly', async () => {
  const generateString = (length) => {
    const specialChars = '!@#$%^&*()_+[]{}|;:,.<>?';
    let result = '';
    for (let i = 0; i < length; i++) {
      if (Math.random() > 0.7) {
        result += specialChars.charAt(Math.floor(Math.random() * specialChars.length));
      } else {
        result += faker.string.alphanumeric(1);
      }
    }
    return result;
  }
  for (let i = 0; i < 10; i++) {
    const randomString = generateString(faker.number.int({ min: 500, max: 1000 }));
    const result = await exec(`bash execute.sh ${randomString}`);
    expect(result?.trim()).toBe(encodeURIComponent(randomString));
  }
});
```

**Key idea:** URL-encode with jq: `printf %s "$1" | jq -sRr @uri`, or `jq -nr --arg s "$1" '$s|@uri'`.

**Solution: team *Byte Bandits***
```bash
#!/bin/bash
if [ -z "$1" ]; then exit 0; fi
echo -n "$1" | jq -sRr @uri
```

---

---

## 2024 · MEDIUM

### Code Forge (2024 · medium)

In an ancient forge, a master blacksmith receives a small Golang script, a raw piece of iron in the world of code. Your task is to transform this script into a finely crafted executable, much like forging a blade from raw metal. Use your expertise to build and shape the script into a functional program, showcasing the artistry and precision of a master coder.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A single executable file named `blade` in a directory named `out`.

- The script must be compiled using Go 1.17.2

**Starter code**

Starter `execute.sh`: (empty)

`src/script.go`:
```go
package main

import "fmt"

func main() {
	fmt.Println("Hello there... Welcome to Bashaway 2k24!")
}
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
test('should build the script into a runnable executable', async () => {
  await exec('bash execute.sh')
  const version = await exec('go version out/blade')
  expect(version?.trim()).toBe('out/blade: go1.17.2')
  const result = await exec('out/blade')
  expect(result?.trim()).toBe('Hello there... Welcome to Bashaway 2k24!')
});
```

**Key idea:** Get an exact Go version with the official Docker image: `docker run --rm -v "$PWD":/w -w /w golang:1.17.2 go build -o out/blade src/script.go`. Don't use `-d` (detached). It lets the script exit before the build finishes. Another way is downloading the go1.17.2 tarball (team DreamGen).

**Solution: team *Team Veritas*** (uses `-d`. Remove it so the build finishes before the test runs.)
```bash
#!/bin/bash
SRC_FILE="src/script.go"
OUT_DIR="out"
OUT_FILE="blade"
mkdir -p $OUT_DIR
docker run -d --rm -v "$PWD":/usr/src/myapp -w /usr/src/myapp golang:1.17.2 go build -o $OUT_DIR/$OUT_FILE $SRC_FILE
```

---

### Cybernetic Frontier (2024 · medium)

Within the dim glow of a cybernetic frontier, a shard of code from the Cybersource Secure Acceptance payment flow beckons. Your mission: resurrect its power on localhost:5000. But in this cyber game of cat and mouse, success means mastering the art of the unseen, navigating the shadows for coins that hold the key to unlocking hidden truths.

**Constraints**

- None

**Output / Evaluation Criteria**

- Get the webapp lying within the `src` directory up and running on the given port by using any means necessary

**Starter code**

Starter `execute.sh`: 
```bash
# Start Install Google Chrome (You may comment out these lines during local testing if you already have Chrome installed. Though the chromedriver installed here might not work with your Chrome version)
sudo apt update
wget https://mirror.cs.uchicago.edu/google-chrome/pool/main/g/google-chrome-stable/google-chrome-stable_126.0.6478.114-1_amd64.deb
sudo dpkg -i google-chrome-stable_126.0.6478.114-1_amd64
sudo apt-get install -y -f
rm google-chrome-stable_126.0.6478.114-1_amd64
# End Install Google Chrome
# Write your code here
```

Extra npm deps used by the tests: `chromedriver@122.0.0, selenium-webdriver@4.12.0`

*Project tree (8 files, 119 KB), shown 2 levels deep:*
```
src/jsp.iml
src/web/jquery-1.7.min.js
src/web/payment.css
src/web/payment_confirmation.jsp
src/web/payment_form.js
src/web/payment_form.jsp
src/web/receipt.jsp
src/web/security.jsp
```

**Test `test/index.test.js`**
```js
const { Browser, Builder, By, until } = require('selenium-webdriver')
const chrome = require('selenium-webdriver/chrome');
require('chromedriver')
jest.setTimeout(10000);
describe('should check if the application is up and running', () => {
  const rootURL = 'http://localhost:5000'
  const chromeOptions = new chrome.Options();
  chromeOptions.addArguments("--no-sandbox")
  chromeOptions.addArguments("--disable-dev-shm-usage")
  chromeOptions.addArguments("--headless")
  const d = new Builder().forBrowser(Browser.CHROME).setChromeOptions(chromeOptions).build()
  const waitUntilTime = 5000
  let driver
  beforeAll(async () => {
    await d.then(async _d => {
      driver = _d
      await driver.get(rootURL)
    })
  })
  afterAll((done) => done())
  const getElementById = async (id) => {
    const el = await driver.wait(until.elementLocated(By.id(id)), waitUntilTime)
    return await driver.wait(until.elementIsVisible(el), waitUntilTime)
  }
  describe('payment form', () => {
    beforeEach(async () => {
      await driver.navigate().to(`${rootURL}/payment_form.jsp`);
    })
    test('payment form should exist', async () => {
      const form = await getElementById('payment_form');
      expect(form).not.toBe(null);
    });
    test('payment details section should exist', async () => {
      const section = await getElementById('paymentDetailsSection');
      expect(section).not.toBe(null);
    });
    test('click default button', async () => {
      const btn = await getElementById('defaultAll')
      await btn.click()
      const transactionTypeInput = await getElementById('transaction-type-input')
      await expect(transactionTypeInput.getAttribute("value")).resolves.toEqual('authorization');
      const amountInput = await getElementById('amount-input')
      await expect(amountInput.getAttribute("value")).resolves.toEqual('100.00');
      const currencyInput = await getElementById('currency-input')
      await expect(currencyInput.getAttribute("value")).resolves.toEqual('USD');
    })
    test('navigate to confirmation on submit', async () => {
      const btn = await getElementById('submit')
      await btn.click()
      const confirmationForm = await getElementById('payment_confirmation')
      expect(confirmationForm).not.toBe(null);
      await expect(driver.getCurrentUrl()).resolves.toContain("payment_confirmation.jsp")
    })
    test('navigate to cybersource on submit and confirm', async () => {
      const btn = await getElementById('submit')
      await expect(btn.getAttribute("value")).resolves.toEqual("Submit");
      await btn.click()
      const confirmBtn = await getElementById('confirm')
      await expect(confirmBtn.getAttribute("value")).resolves.toEqual("Confirm");
      await confirmBtn.click()
      await expect(driver.getCurrentUrl()).resolves.toContain("testsecureacceptance.cybersource.com")
    })
  })
})
```

**Key idea:** This is a legacy JSP app. Run it in a `tomcat` container with `src/web` copied to `webapps/ROOT` and map host port 5000 to 8080. The Chrome install lines in the starter are for the test's Selenium browser.

**Solution: repo contributor**
```bash
sudo apt update
wget https://mirror.cs.uchicago.edu/google-chrome/pool/main/g/google-chrome-stable/google-chrome-stable_126.0.6478.114-1_amd64.deb
sudo dpkg -i google-chrome-stable_126.0.6478.114-1_amd64
sudo apt-get install -y -f
rm google-chrome-stable_126.0.6478.114-1_amd64
docker build -t cybersource .
docker run -d -p 5000:8080 cybersource
```
`dockerfile`:
```dockerfile
FROM tomcat:7

COPY ./src/web /usr/local/tomcat/webapps/ROOT

CMD ["catalina.sh", "run"]
```

---

### Dot Detective (2024 · medium)

In the sprawling digital metropolis of Data City, a rogue data scientist has encrypted vital information within a massive JSON file, nesting keys within keys, hiding the clues to a grand mystery. Your mission, should you choose to accept it, is to unravel this intricate web and convert the convoluted JSON structure into a simplified dot notation format.

Your task: Write a program that takes a large, deeply nested JSON file with random keys and transforms it into the following format:

```json
{
  "a.b": "c",
  "x.y.z": "value"
}
```

This challenge isn't for the faint-hearted. Only the best will decode the JSON labyrinth and restore order to the data chaos. Are you ready to become the Dot Detective of Data City?

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A single file named **transformed.json** should be created in a **out** directory which has the transformed content of the given JSON file

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `@types/dot-object@2.1.6, dot-object@2.1.5`

`src/chaos.json`:
```json
{
  "eum": {
    "doloremque": "Ducimus optio cupiditate.",
    "maxime": "Nobis itaque rerum voluptatum quidem ex sint officiis amet corrupti.",
    "totam": "Rerum reprehenderit tempora assumenda quasi magni autem animi corporis."
  },
  "id": {
    "esse": "Iste animi at molestias odio assumenda voluptatibus nihil sequi.",
    "ad": "Veritatis earum deleniti facilis culpa corrupti cumque."
  }
}
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const dot = require('dot-object');
jest.setTimeout(15000);
test('should not download external scripts', () => {
  const script = fs.readFileSync('./execute.sh', 'utf-8')
  expect(script).not.toContain("curl");
  expect(script).not.toContain("wget");
  expect(script).not.toContain("fetch");
  expect(script).not.toContain("git clone");
});
test('should check if the files are merged correctly', async () => {
  function generateChaos(depth, breadth) {
    if (depth === 0) return faker.lorem.sentence();
    let obj = {};
    for (let i = 0; i < breadth; i++) {
      if (faker.number.int({ min: 0, max: 2 })) {
        obj[faker.lorem.word()] = generateChaos(depth - 1, breadth);
      }
    }
    return obj;
  }
  const chaos = generateChaos(faker.number.int({ min: 2, max: 4 }), faker.number.int({ min: 1, max: 4 }))
  fs.writeFileSync('./src/chaos.json', JSON.stringify(chaos, null, 2));
  if (fs.existsSync('./out')) fs.rmdirSync('./out', { recursive: true });
  await exec('bash execute.sh');
  const transformed = fs.readFileSync('./out/transformed.json', 'utf-8');
  expect(JSON.parse(transformed)).toMatchObject(dot.dot(chaos));
});
```

**Key idea:** Flatten JSON with jq's `paths(scalars)`: `[paths(scalars) as $p | {key: ($p|join(".")), value: getpath($p)}] | from_entries`. It's one line, compared with recursive bash functions.

**Solution: team *Tux***
```bash
#!/usr/bin/env bash
mkdir -p out
jq '[paths(scalars) as $path | {"key": $path | join("."), "value": getpath($path)}] | from_entries' src/chaos.json > out/transformed.json
```

---

### Eavesdropper (2024 · medium)

In a tale of intrigue and whispers, an eavesdropper embarks on a mischievous quest. Summon forth a Kubernetes cluster upon your local machine, and conjure the Metrics API to uncover the hidden truths it guards. Demonstrate your cunning in this intricate dance of metrics and subterfuge, where each data point is a glimpse into the unseen.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- A local Kubernetes cluster should be running

- Cluster should be set as the current context

- Metrics APIs should be available

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
test('a local cluster must be running with the metrics server installed', async () => {
  const result = await exec('kubectl top nodes');
  expect(result).toContain('NAME');
  expect(result).toContain('CPU');
  expect(result).toContain('MEMORY');
});
```

**Key idea:** `minikube start` then `minikube addons enable metrics-server`. Wait until `kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes` succeeds. On other clusters (k3d, kind) apply the metrics-server `components.yaml`.

**Solution: team *KK_Coders***
```bash
#!/bin/bash
minikube start  --wait=all --driver=docker
kubectl config use-context minikube
minikube addons enable metrics-server
echo "Waiting for metrics-server to be up..."
while ! kubectl get apiservices | grep -q 'v1beta1.metrics.k8s.io.*True'; do
    sleep 1
done
while ! kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes" >/dev/null 2>&1; do
  echo "Metrics API not available yet, retrying in 5 seconds..."
  sleep 5
done
echo "Metrics API is available!"
while true; do
    if kubectl get pods -n kube-system | grep metrics-server | grep Running >/dev/null; then
        echo "Metrics Server is running!"
        break
    else
        echo "Waiting for Metrics Server to be ready..."
        sleep 5
    fi
done
sleep 90
kubectl top nodes
```

---

### Enchanted Artifacts (2024 · medium)

In a mystical forge, a blacksmith named Orin crafts enchanted artifacts using secret keys that unlock elemental powers. These keys which are provided as **build arguments vary with each creation** and must be embedded into the artifact's blueprint during the forging process.

An example of a list of required keys (which will be generated during tests) and the corresponding build command to its artifact are as follows:

```.env.example
KEY1=
KEY2=
```

```bash
docker build --build-arg KEY1=fire --build-arg KEY2=water -t bashaway-2k24-enchanted-artifact .
```

If you were Orin, how would you inscribe these ever-changing keys into the artifact's blueprint, ensuring they are seamlessly integrated and activated during the forging process?

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

- You are allowed to make changes to the Dockerfile

**Output / Evaluation Criteria**

- The given express server should be able to be built with the given Dockerfile and run successfully. It's root route should all key values as a JSON object (These key values should be the same as the ones in the .env file and are picked from the container environment)

- The script should not execute if the .env.example file is missing

**Starter code**

Starter `execute.sh`: (empty)

`src/Dockerfile`:
```dockerfile
FROM oven/bun:1.1.15-alpine AS runner

COPY . .

CMD ["bun", "run", "server.js"]
```

`src/server.js`:
```js
import * as fs from 'fs';

Bun.serve({
    port: 3000,
    fetch() {
        const envExampleLines = fs.readFileSync('.env.example', 'utf8').split('\n');
        const keys = envExampleLines.reduce((acc, line) => {
            const [key] = line.split('=');
            acc[key] = process.env[key];
            return acc;
        }, {});
        return new Response(JSON.stringify(keys), {
            headers: {
                'Content-Type': 'application/json'
            }
        });
    },
});
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
jest.setTimeout(15000);
test('should not download external scripts', () => {
  const script = fs.readFileSync('./execute.sh', 'utf-8')
  expect(script).not.toContain("curl");
  expect(script).not.toContain("wget");
  expect(script).not.toContain("fetch");
  expect(script).not.toContain("git clone");
});
describe('should if the .env files are generated', () => {
  const keys = {};
  beforeAll(async () => {
    for (let i = 0; i < faker.number.int({ min: 10, max: 25 }); i++) {
      keys[faker.word.noun().toUpperCase()] = faker.system.fileName();
    }
    fs.writeFileSync(`./src/.env.example`, Object.keys(keys).map(key => `${key}=`).join('\n'));
    await exec('bash execute.sh');
  });
  test('docker image should build and run with the correct environment variables', async () => {
    const tag = 'bashaway-2k24-enchanted-artifact'
    process.chdir("./src");
    await exec(`docker build ${Object.entries(keys).map(([key, value]) => `--build-arg ${key}=${value}`).join(' ')} -t ${tag} .`);
    await exec(`docker run -d -p 3000:3000 --name=${tag} ${tag}`)
    await expect(exec(`docker ps -f name=${tag} --format "{{.Names}}-{{.Ports}}"`)).resolves.toContain(`${tag}-0.0.0.0:3000->`);
    await new Promise(resolve => setTimeout(resolve, 3000));
    const result = await exec(`curl http://localhost:3000`);
    expect(JSON.parse(result)).toMatchObject(keys);
  });
});
```

**Key idea:** Generate the Dockerfile from `.env.example`. For every key write `ARG KEY` and `ENV KEY=${KEY}`, so build args become runtime env vars. Exit early if `.env.example` is missing.

**Solution: team *5H4RK***
```bash
#!/bin/bash
if [ ! -f "./src/.env.example" ]; then
  echo ".env.example file is missing. Exiting."
  exit 0
fi
ARG_LINES=""
ENV_LINES=""
while IFS= read -r line || [[ -n $line ]]; do
    KEY=$(echo "$line" | cut -d '=' -f 1)
    ARG_LINES+="ARG $KEY\n"
    ENV_LINES+="ENV $KEY=\${$KEY}\n"
done < ./src/.env.example
{
  echo "FROM oven/bun:1.1.15-alpine AS runner"
  echo ""
  echo "COPY . ."
  echo ""
  echo -e "$ARG_LINES"
  echo -e "$ENV_LINES"
  echo ""
  echo "EXPOSE 3000"
  echo ""
  echo "CMD [\"bun\", \"run\", \"server.js\"]"
} > ./src/Dockerfile
```

---

### Frugal Villager (2024 · medium)

In a humble village, a resourceful farmer needs to save every coin. He decides to plant his own crops instead of buying expensive ones from the market. In this spirit, you must do the same in the realm of technology.

On your Kubernetes homestead, cultivate a service named after a reliable horse, "redis," and let it roam freely in the "bashaway" pasture. Ensure this service can be reached through the village gate at port 6381. Share the steps to achieve this while maintaining the thriftiness and efficiency akin to a frugal villager.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- A local Kubernetes cluster should be running

- Cluster should be set as the current context

- A service named "redis" should be running in the "bashaway" namespace exposed at port 6381

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `ioredis@5.2.3`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
const Redis = require('ioredis').default;
jest.setTimeout(15000);
const port = "6381"
test('a service must be running with the name redis', async () => {
  const services = await exec('kubectl get svc -n bashaway');
  expect(services).toContain('redis');
  const forwardedPorts = await exec(`lsof -iTCP:${port} -sTCP:LISTEN -P`);
  expect(forwardedPorts).toContain(port);
});
test('must connect to the redis server', async () => {
  const password = await exec('kubectl get secret redis -n bashaway -o jsonpath="{.data.redis-password}"');
  const decodedPassword = Buffer.from(password, 'base64').toString('utf-8').trim();
  const redis = new Redis(`redis://:${decodedPassword}@127.0.0.1:${port}`);
  const ping = await redis.ping();
  expect(ping).toBe('PONG');
  redis.disconnect();
});
```

**Key idea:** Start a cluster, `kubectl create ns bashaway`, deploy redis with a Helm chart or raw YAML, then `kubectl port-forward svc/redis 6381:6379 -n bashaway &`. The test needs a service named exactly `redis`. Note that the Helm chart names it `redis-master`.

**Solution: team *Byte Bandits*** (raw Deployment YAML via heredoc + `kubectl expose`)
```bash
#!/bin/bash
minikube start
kubectl config use-context minikube
kubectl create namespace bashaway
REDIS_PASSWORD=$(openssl rand -base64 20)
kubectl create secret generic redis --from-literal=redis-password=$REDIS_PASSWORD -n bashaway
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: bashaway
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis
        args: ["--requirepass", "$(REDIS_PASSWORD)"]
        env:
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: redis
              key: redis-password
        ports:
        - containerPort: 6379
EOF
kubectl expose deployment redis --name=redis --port=6379 --target-port=6379 -n bashaway
kubectl wait --for=condition=ready pod -l app=redis -n bashaway --timeout=60s
kubectl port-forward service/redis 6381:6379 -n bashaway &
sleep 5
kubectl get services -n bashaway
echo "Redis password: $REDIS_PASSWORD"
```

**Solution: repo contributor** (Helm version (shorter, check the service name))
```bash
minikube start
kubectl create ns bashaway
helm install redis oci://registry-1.docker.io/bitnamicharts/redis -n bashaway
sleep 45
kubectl port-forward svc/redis-master 6381:6379 -n bashaway &
```

---

### Gearsmith (2024 · medium)

Within the hallowed workshop of a master gearsmith, two intricate gear manuscripts were unearthed. The first, inscribed as **gears.js**, details the initial schematics. The second, marked **gears-1719135696714**.js, reveals the refinements and enhancements made to the original.

Your task is to **craft a detailed manuscript (patch file)** that encapsulates the precise alterations made between these two versions. This manuscript must be capable of converting the initial design (gears.js) into the refined version (gears-1719135696714.js) and also allow the reverse transformation to restore the original design.

Employ your expertise to produce this transformative manuscript, ensuring it captures the modifications with precision and can be utilized for future iterations in the gearsmith's legacy.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

- The script must be **less than 100 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- A single patch file named **gearsmith.patch** in a directory named **out** that can be applied to the initial design (gears.js) to produce the refined version (gears-1719135696714.js) and vice versa.

**Starter code**

Starter `execute.sh`: (empty)

`src/gears-1719135696714.js` (1500 bytes, 48 lines, first 8 lines shown):
```js
let gearTeeth = 20;
let gearRatio = 2.5;
let gearSpeed = 150; // RPM
let gearDiameter = 10; // cm

console.log('Gear setup initialized.');

function calculateCircumference(diameter) {
...
```

`src/gears.js`:
```js
let gearTeeth = 20;
let gearRatio = 2.5;
let gearSpeed = 150; // RPM
let gearDiameter = 10; // cm

function calculateCircumference(diameter) {
  let circumference = Math.PI * diameter;
  console.log('Circumference calculated:', circumference);
  return circumference;
}

function calculateDrivenSpeed(driverSpeed, ratio) {
  let drivenSpeed = driverSpeed / ratio;
  console.log('Driven speed calculated:', drivenSpeed);
  return drivenSpeed;
}

function calculateGearRatio(teeth1, teeth2) {
  let ratio = teeth1 / teeth2;
  console.log('Gear ratio calculated:', ratio);
  return ratio;
}

let circumference = calculateCircumference(gearDiameter);
let drivenSpeed = calculateDrivenSpeed(gearSpeed, gearRatio);
let calculatedRatio = calculateGearRatio(gearTeeth, gearTeeth / gearRatio);

console.log('Gear calculations complete.');
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 100 characters in length', () => {
    expect(script.length).toBeLessThan(100);
  });
});
test('should not have tampered with the original files', () => {
  const gearsSize = fs.statSync('src/gears.js').size;
  expect(gearsSize).toBe(829);
  const gearsTemporarySize = fs.statSync('src/gears-1719135696714.js').size;
  expect(gearsTemporarySize).toBe(1500);
});
test('patch file should be present', () => {
  expect(fs.existsSync('out/gearsmith.patch')).toBeTruthy();
});
test('should apply patch successfully', async () => {
  process.chdir('src');
  await exec('patch -i ../out/gearsmith.patch');
  const consoleOutput = await exec('node gears.js');
  [
    'Gear setup initialized.',
    'Calculating circumference for diameter: 10',
    'Circumference calculated',
    'Calculating driven speed with driverSpeed: 150 and ratio: 2.5',
    'Driven speed calculated: 60',
    'Calculating gear ratio with teeth1: 20 and teeth2: 8',
    'Gear ratio calculated: 2.5',
    'Starting example usage.',
    'Circumference of the gear',
    'Speed of the driven gear: 60',
    'Calculated gear ratio: 2.5',
    'Gear calculations complete.',
    'End of script.'
  ].forEach(text => {
    expect(consoleOutput).toContain(text);
  });
});
```

**Key idea:** `mkdir -p out; diff -u src/gears.js src/gears-1719135696714.js > out/gearsmith.patch || true`. `diff` exits with 1 when the files differ, so add `|| true` or the run counts as a failure. `patch -R` applies the patch in reverse.

---

### Historian (2024 · medium)

A diligent historian is tasked with uncovering the past of a mysterious machine. His mission is to write a script that, when given a number, reveals the number of commands previously executed on an enigmatic device. However, there's a twist: the historical commands and the script reside in separate worlds **(processes)**, necessitating synchronization through an elusive history file. Wear the shoes of this historian and craft this script that bridges these worlds

**Example Invokation**

```bash
bash execute.sh N
```

- N is the number of historical commands to be displayed

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- The script should output a list of the last N commands executed on the machine, where N is the number provided as an argument.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
const promisify = require('util').promisify;
const exec = promisify(require('child_process').exec);
const execute = async (command) => {
  const { stdout, stderr } = await exec(command);
  if (stderr) throw new Error(stderr);
  return stdout;
};
test('should check for correct output', async () => {
  const count = faker.number.int({ min: 5, max: 15 });
  for (let i = 0; i < count; i++) {
    await execute(`echo "echo ${i}" >> ~/.bash_history`);
  }
  const randomNumber = faker.number.int({ min: 1, max: count });
  const history = await execute(`bash execute.sh ${randomNumber + 1}`);
  for (let i = 0; i < randomNumber; i++) {
    expect(history).toContain(`echo ${count - i - 1}`);
  }
});
```

**Key idea:** The script runs in a new non-interactive shell, so its in-memory history is empty. Read the file directly: `tail -n "$1" ~/.bash_history` (team 5H4RK). The other way is `history -r ~/.bash_history; history "$1"`.

---

### Old Wives (2024 · medium)

In a quaint village, where old wives gather to eavesdrop on whispered tales, a task has surfaced. You must set up a Jaeger instance to capture the secrets of a simple Node.js service and ensure that the environment variables JEAGER_COLLECTOR_ENDPOINT and JAEGER_QUERY_ENDPOINT are properly configured. The old wives depend on your skill to keep the flow of whispered logs intact.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- The logs from the service must be captured by the Jaeger instance

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `@types/express@4.17.13, axios@1.5.0, express@4.19.2, jaeger-client@3.17.2, opentracing@^0.14.4`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const express = require('express');
const axios = require('axios').default;
const { initTracer } = require("jaeger-client");
const { Tags, FORMAT_HTTP_HEADERS } = require('opentracing');
describe('should check if the primary requirements are met', () => {
  let server, tracer;
  const responseKey = faker.string.uuid();
  const randomPort = faker.number.int({ min: 3000, max: 5000 });
  let traceID = ""
  let error = false;
  beforeAll(async () => {
    tracer = initTracer({
      serviceName: 'my-awesome-service',
      sampler: {
        type: "const",
        param: 1,
      },
      reporter: {
        collectorEndpoint: `${process.env.JEAGER_COLLECTOR_ENDPOINT}/api/traces`,
        logSpans: true,
      },
    }, {
      logger: {
        info(msg) {
          traceID = msg.split(" ")[2].split(":")[0]?.trim();
          console.log("INFO ", msg);
        },
        error(msg) {
          error = true;
          console.log("ERROR", msg);
        },
      },
    });
    const app = express();
    app.get('/', (req, res) => {
      const span = tracer.startSpan('root-span');
      const requestId = Date.now().toString();
      span.setTag(Tags.HTTP_URL, req.url);
      span.setTag(Tags.HTTP_METHOD, req.method);
      span.setTag(Tags.SPAN_KIND, Tags.SPAN_KIND_RPC_CLIENT);
      span.setTag("requestId", requestId);
      span.log({ event: 'request-received' });
      tracer.inject(span, FORMAT_HTTP_HEADERS, req.headers);
      res.send(responseKey);
      span.log({ event: 'response-sent', responseKey });
      span.finish();
    });
    server = app.listen(randomPort);
  });
  test('response should be equal to the response key', async () => {
    const response = await axios.get(`http://localhost:${randomPort}`);
    expect(response.data).toBe(responseKey);
  });
  test('trace ID should be present in the logs', async () => {
    await new Promise(resolve => setTimeout(resolve, 2500));
    const response = await axios.get(`${process.env.JAEGER_QUERY_ENDPOINT}/api/traces/${traceID}?prettyPrint=true`);
    expect(response.data.data[0].traceID).toBe(traceID);
    expect(response.data.data[0].spans[0].logs[1].fields[1].key).toBe("responseKey");
    expect(response.data.data[0].spans[0].logs[1].fields[1].value).toBe(responseKey);
  });
  test('there should be no errors', () => {
    expect(error).toBe(false);
  });
  afterAll(async () => {
    await server.close();
    await tracer.close();
  });
});
```

**Key idea:** Run `jaegertracing/all-in-one` in Docker. To pass env vars to later steps in GitHub Actions, append `KEY=value` to `$GITHUB_ENV`, because a plain `export` dies with the script.

**Solution: repo contributor**
```bash
echo "JEAGER_COLLECTOR_ENDPOINT=http://localhost:14268" >> $GITHUB_ENV
echo "JAEGER_QUERY_ENDPOINT=http://localhost:16686" >> $GITHUB_ENV
docker run -d --name jaeger -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 -p 5775:5775/udp -p 6831:6831/udp -p 6832:6832/udp -p 5778:5778 -p 16686:16686 -p 14268:14268 -p 9411:9411 jaegertracing/all-in-one:1.8
sleep 10
```

---

### Sealed Whispers (2024 · medium)

Attend, O brave soul, to this spectral tale. In the haunted lands where wraiths whisper and phantoms guard their secrets, a mighty task awaits. Thou must prepare an environment with a Kubernetes cluster, imbued with the arcane powers of Bitnami’s Sealed Secrets.

With these spectral tools, thou shalt take an ordinary Kubernetes secret and encase it within an ethereal seal, encrypting it against all mortal prying. Apply this sealed whisper to the cluster, ensuring its hidden truths are revealed only to those with the rightful incantation.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- A local Kubernetes cluster should be running

- Cluster should be set as the current context

- Should be able to encrypt a secret using kubeseal and apply it to the cluster

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `js-yaml@4.1.0`

`src/secret.yml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: auth-service-secret
stringData:
  PORT: "8080"
  JWT_SECRET: "vfcd43d"
  REDIS_CONNECTION_STRING: "redis://:iusfdnhd3@redis-master:6379"
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
const yaml = require('js-yaml');
test('should encrypt and apply the secret successfully', async () => {
  if (fs.existsSync('./out')) {
    fs.rmSync('./out', { recursive: true });
  }
  fs.mkdirSync('./out');
  await exec('bash -c "cat ./src/secret.yml | kubeseal --format yaml --controller-namespace kube-system --controller-name sealed-secrets > out/sealed-secret.yml"');
  await expect(exec('bash -c "kubectl get secrets/auth-service-secret"')).rejects.toThrow();
  await exec('kubectl apply -f ./out/sealed-secret.yml');
  const sealedSecretYml = fs.readFileSync('./out/sealed-secret.yml', 'utf-8');
  const sealedSecret = yaml.load(sealedSecretYml);
  expect(sealedSecret.kind).toBe('SealedSecret');
  expect(sealedSecret.metadata.name).toBe('auth-service-secret');
  const secret = await exec('bash -c "kubectl get secrets/auth-service-secret --template={{.data.JWT_SECRET}}"')
  expect(Buffer.from(secret, 'base64').toString()).toBe('vfcd43d');
});
```

**Key idea:** Install the sealed-secrets controller with Helm and install the `kubeseal` CLI binary (no teams solved this one).

**Solution: repo contributor (untested)**
```bash
minikube start
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets sealed-secrets/sealed-secrets --namespace kube-system --version 2.16.1
KUBESEAL_VERSION='0.27.1'
curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION:?}/kubeseal-${KUBESEAL_VERSION:?}-linux-amd64.tar.gz"
tar -xvzf kubeseal-${KUBESEAL_VERSION:?}-linux-amd64.tar.gz kubeseal
rm kubeseal-${KUBESEAL_VERSION:?}-linux-amd64.tar.gz
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
rm kubeseal
sleep 10
```

---

### Serpentine Paths (2024 · medium)

In the ancient realms where the Nagas dwell, a task of great importance arises. Spin up a local Firebase Firestore instance, a repository of countless data points, so that the given code may weave its path to connect seamlessly. The serpentine paths of data await your guidance.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- A local Firebase Firestore instance should be running on `localhost:9000` after the script is executed.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `firebase@10.12.5`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const { initializeApp } = require('firebase/app');
const { getFirestore, collection, doc, getDoc, setDoc, connectFirestoreEmulator, terminate } = require('firebase/firestore');
test('should connect to firestore and add a document', async () => {
  initializeApp({
    projectId: 'test',
  });
  const db = getFirestore();
  connectFirestoreEmulator(db, 'localhost', 9000);
  const collectionRef = collection(db, 'test');
  const docRef = doc(collectionRef, 'test-doc');
  const value = Math.random().toString(36).substring(7);
  await setDoc(docRef, { test: value });
  const docSnap = await getDoc(docRef);
  expect(docSnap.exists()).toBe(true);
  expect(docSnap.data().test).toBe(value);
  terminate(db);
});
```

**Key idea:** Run a Firestore emulator in Docker (`mtlynch/firestore-emulator-docker`, port 9000:8080). The alternatives are `firebase emulators:start --only firestore` with a `firebase.json`, or `gcloud emulators firestore start --host-port=localhost:9000`.

**Solution: team *Red Shifts***
```bash
#!/bin/bash
docker run -d \
  --name test \
  --env "FIRESTORE_PROJECT_ID=test" \
  --env "PORT=8080" \
  --publish 9000:8080 \
  mtlynch/firestore-emulator-docker
```

---

---

## 2024 · HARD

### Nefarious Threats (2024 · hard)

Hark, noble coder, a tale of valor unfolds before thee. In the vast kingdom of Databasia, a precious MongoDB server must be summoned upon thy local machine, its port numbered 27020. The realm demands that this server be fortified with the armor of TLS connections, shielding it from the nefarious threats lurking in the shadows.

To achieve this, thou must inscribe within the sacred environment variables MONGO_TLS_CA_FILE_PATH and MONGO_TLS_CERT_KEY_PATH the paths to the revered TLS certificate authority file and a certificate key file, respectively. Take up this quest with the utmost precision, ensuring the sanctity and security of the server remains unbreached. The kingdom's trust lies in thy skilled hands.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- A mongodb server must be running on port 27020 with TLS enabled and connections to it must succeed.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `mongodb@6.8.0`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const mongodb = require("mongodb")
jest.setTimeout(10000);
test('properly configured server should be up and running', async () => {
  const client = new mongodb.MongoClient(`mongodb://localhost:27020/test`, {
    tls: true,
    tlsCAFile: process.env.MONGO_TLS_CA_FILE_PATH,
    tlsCertificateKeyFile: process.env.MONGO_TLS_CERT_KEY_PATH,
  });
  await client.connect();
  const db = client.db('bashaway');
  const randomString = Math.random().toString(36).substring(7);
  await db.collection('test').insertOne({ test: randomString });
  const insertedRow = await db.collection('test').findOne({ test: randomString });
  expect(insertedRow).toMatchObject({ test: randomString });
  await client.close();
});
```

**Key idea:** TLS for MongoDB: create a CA with openssl and sign a server cert for CN=localhost. `cat key crt > mongodb.pem`, then run mongod with `--tlsMode requireTLS --tlsCertificateKeyFile ... --tlsCAFile ...`. The Docker version is `docker run -p 27020:27017 -v $PWD/certs:/c mongo --tlsMode requireTLS ...`. Export the two paths through `$GITHUB_ENV`.

**Solution: repo contributor**
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem -subj "/C=LK/ST=/L=/O=/OU=/CN="
openssl genrsa -out mongodb.key 2048
openssl req -new -key mongodb.key -out mongodb.csr -subj "/C=LK/ST=/L=/O=/OU=/CN=localhost"
openssl x509 -req -in mongodb.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out mongodb.crt -days 500 -sha256
cat mongodb.key mongodb.crt > mongodb.pem
sudo mv mongodb.pem /etc/ssl/mongodb.pem
sudo mv rootCA.pem /etc/ssl/rootCA.pem
rm -f mongodb.key mongodb.crt mongodb.csr rootCA.key rootCA.srl
sudo sed -i '/^net:/a\ \ ipv6: true\n\ \ ssl:\n\ \ \ \ mode: requireSSL\n\ \ \ \ PEMKeyFile: /etc/ssl/mongodb.pem\n\ \ \ \ CAFile: /etc/ssl/rootCA.pem' /etc/mongod.conf
sudo sed -i 's/bindIp: 127.0.0.1/bindIp: 127.0.0.1, ::1/' /etc/mongod.conf
sudo sed -i 's/27017/27020/g' /etc/mongod.conf
echo "MONGO_TLS_CA_FILE_PATH=/etc/ssl/rootCA.pem" >> $GITHUB_ENV
echo "MONGO_TLS_CERT_KEY_PATH=/etc/ssl/mongodb.pem" >> $GITHUB_ENV
sudo systemctl restart mongod
sudo systemctl status mongod
```

---

### Noble Endeavor (2024 · hard)

In the age of quills and ink, thou art summoned to a grand task. A sample SQL scroll dost hold a table of data, awaiting thy hand to transport it to a MongoDB collection of the same name. The MongoDB server, secured with a password 3aDAl3vfeRbY1lf, lies upon port 27020. Pen a script that deftly performs this migration, ensuring that not a single byte is lost in transit. Show thy mastery in the art of data, and let no hindrance mar this noble endeavor.

**Constraints**

- None

**Output / Evaluation Criteria**

- A MongoDB server should be running on port 27020.

- There should be a database with the same name as the one in the SQL scroll.

- The database should contain a collection with the same name as the table in the SQL scroll.

- The collection should contain all the data from the SQL scroll.

- The script should not execute if the SQL scroll is missing

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `mongodb@6.8.0, mysql2@3.10.2`

**Test `test/index.test.js`**
```js
const mongodb = require("mongodb")
jest.setTimeout(300000);
test('should migrate the data from the sql file to a mongodb database', async () => {
  if (fs.existsSync('./src')) {
    fs.rmdirSync('./src', { recursive: true });
  }
  fs.mkdirSync('./src');
  const database = faker.vehicle.model()?.split(' ')?.[0]?.toLowerCase();
  const table = faker.lorem.word();
  const columns = new Set();
  for (let i = 0; i < faker.number.int({ min: 10, max: 25 }); i++) {
    const c = faker.database.column();
    if (c != "id") columns.add(c);
  }
  const rows = faker.number.int({ min: 10, max: 10 });
  const data = Array.from({ length: rows }, () => {
    return Array.from(columns).reduce((acc, column) => {
      acc[column] = faker.number.int({ min: 0, max: 100 });
      return acc;
    }, {});
  });
  const sql = `CREATE DATABASE ${database};
  USE ${database};
  CREATE TABLE ${table} (${Array.from(columns).map(column => `\`${column}\` INT`).join(', ')});
  INSERT INTO ${table} VALUES ${data.map(row => `(${Object.values(row).join(', ')})`).join(', ')};`;
  const scrollName = faker.lorem.word();
  fs.writeFileSync(`./src/scroll-${scrollName}.sql`, sql);
  await exec('bash execute.sh');
  const client = new mongodb.MongoClient(`mongodb://admin:3aDAl3vfeRbY1lf@localhost:27020/${database}?directConnection=true`);
  await client.connect();
  const db = client.db(database);
  const insertedRows = await db.collection(table).find().toArray();
  expect(insertedRows).toHaveLength(rows);
  insertedRows.forEach((row, i) => {
    expect(row).toMatchObject(data[i]);
  });
  await client.close();
});
```

**Key idea:** Parse the SQL with grep/sed (DB name, table, columns, VALUES tuples). Start mongo in Docker on port 27020 and create user `admin` **on that database**, because the test connects with `/<db>` as the auth source. Insert the rows with `mongosh`. The test always makes exactly 10 rows of INT columns. A cleaner route is to load the SQL into a MySQL container, `SELECT ... INTO` JSON/CSV, and `mongoimport`.

**Solution: team *Red Shifts*** (pure-bash SQL parsing. It's verbose but it shows the whole flow)
```bash
#! /bin/bash
if [ ! -f src/scroll-*.sql ]; then
    echo "File not found"
    exit 0
fi
FILE_NAME=$(find src -maxdepth 1 -type f -name "scroll-*")
SQL=$(cat $FILE_NAME)
DB_NAME=$(echo "$SQL" | head -n 1 | tail -n 1 | awk '{print $3}' | tr -d ';')
TABLE_NAME=$(echo "$SQL" | head -n 3 | tail -n 1 | awk '{print $3}')
COLUMNS=$(echo "$SQL" | head -n 3 | tail -n 1 | sed -n 's/.*(\(.*\));/\1/p' | tr ', ' ' ' | sed 's/`//g')
read -a COLUMN_ARRAY <<< $(echo "$COLUMNS" | sed 's/INT//g; s/ *$//; s/^ *//; s/  */ /g')
VALUES=$( echo "$SQL" | sed -n 's/.*VALUES (\(.*\));/\1/p')
for row in "${COLUMN_ARRAY[@]}"; do
    echo "$row"
done
IFS=', ' read -r -a rows <<< $(echo $VALUES | tr -d '()')
row_count=$((${#rows[@]} / ${#COLUMN_ARRAY[@]}))
echo "Row count is: "
echo $row_count
echo "Column count is: "
echo ${COLUMN_ARRAY[@]}
echo ${#COLUMN_ARRAY[@]}
echo $TABLE_NAME
MONGO_DATA_DIR="/tmp/mongodata"
MONGO_LOG_FILE="/tmp/mongolog"
MONGO_HOST="localhost"
MONGO_PORT="27020"
DB_NAME=$DB_NAME
USERNAME="admin"
PASSWORD="3aDAl3vfeRbY1lf"
COLLECTION_NAME=$TABLE_NAME
mkdir -p $MONGO_DATA_DIR
docker run -d -p $MONGO_PORT:27017 --name noble mongo
if [ $? -ne 0 ]; then
    echo "Failed to start MongoDB. Check the log file at $MONGO_LOG_FILE"
    exit 0
fi
mongosh --port $MONGO_PORT <<EOF
use $DB_NAME
db.createUser({
  user: "$USERNAME",
  pwd: "$PASSWORD",
  roles: [{ role: "dbOwner", db: "$DB_NAME" }]
})
EOF
if [ $? -eq 0 ]; then
    echo "User created successfully with dbOwner role for $DB_NAME database."
else
    echo "Failed to create user."
    exit 0
fi
echo "MongoDB started successfully."
mongosh --port $MONGO_PORT -u $USERNAME -p $PASSWORD --authenticationDatabase $DB_NAME << EOF
use $DB_NAME
db.createCollection("$COLLECTION_NAME")
EOF
for i in {0..9}
do
    row_output="{"
    for j in "${!COLUMN_ARRAY[@]}"
    do
        value="${rows[$((i * ${#COLUMN_ARRAY[@]} + j))]}"
        row_output+="'${COLUMN_ARRAY[$j]}': $value"
        if [ $j -lt $((${#COLUMN_ARRAY[@]} - 1)) ]; then
            row_output+=$', '
        fi
    done
    row_output+="}"
    printf "%s\n" "$row_output"
    mongosh --port $MONGO_PORT -u $USERNAME -p $PASSWORD --authenticationDatabase $DB_NAME << EOF
    use $DB_NAME
    db.$COLLECTION_NAME.insertOne($row_output)
EOF
done
```

---

### Server Seas (2024 · hard)

Arrr! In the vast seas o' server management, yer mission be to set up a load balancer on port 11000. This balancer'll divvy up incoming requests 'twixt three identical servers, sailin' on ports 11001, 11002 and 11003. Show yer mettle in the art o' load balancin' by configurin' this crew with precision, keepin' the ship runnin' smooth and fair for all who come aboard.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- HTTP requests to the load balancer must be distributed evenly between the given servers.

- The script should only execute if the three servers are running on the specified ports.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `@types/express@4.17.13, axios@1.5.0, express@4.19.2`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const express = require('express');
const axios = require('axios').default  ;
jest.setTimeout(15000)
test('should check if the requests are forwarded properly', async () => {
  const responseStr = faker.string.alphanumeric(64);
  const server1 = express();
  server1.get('/', (_, res) => {
    res.setHeader('x-host-port', 'localhost:11001');
    res.send(responseStr);
  });
  const startedServer1 = await new Promise((resolve) => {
    const server = server1.listen(11001, async () => {
      resolve(server);
    });
  })
  const server2 = express();
  server2.get('/', (_, res) => {
    res.setHeader('x-host-port', 'localhost:11002');
    res.send(responseStr);
  });
  const startedServer2 = await new Promise((resolve) => {
    const server = server2.listen(11002, async () => {
      resolve(server);
    });
  })
  const server3 = express();
  server3.get('/', (_, res) => {
    res.setHeader('x-host-port', 'localhost:11003');
    res.send(responseStr);
  })
  const startedServer3 = await new Promise((resolve) => {
    const server = server3.listen(11003, async () => {
      resolve(server);
    });
  })
  try {
    const err = await axios.get('http://localhost:11000').catch(e => "Error");
    expect(err).toBe("Error");
    await exec(`bash execute.sh`);
    let server1Count = 0;
    let server2Count = 0;
    let server3Count = 0;
    for (let i = 0; i < 100; i++) {
      const response = await axios.get('http://localhost:11000');
      expect(response.data).toBe(responseStr);
      if (response.headers['x-host-port'] === 'localhost:11001') {
        server1Count++;
      } else if (response.headers['x-host-port'] === 'localhost:11002') {
        server2Count++;
      } else if (response.headers['x-host-port'] === 'localhost:11003') {
        server3Count++;
      }
    }
    expect(server1Count).toBeGreaterThan(0);
    expect(server2Count).toBeGreaterThan(0);
    expect(server3Count).toBeGreaterThan(0);
  } catch (e) {
    throw e
  } finally {
    startedServer1.close();
    startedServer2.close();
    startedServer3.close();
  }
});
```

**Key idea:** Check that each backend is up (`lsof -i:PORT`, `nc -z localhost PORT`). Then write an nginx `upstream` block, which is round-robin by default, and `proxy_pass` to it, or use HAProxy with `balance roundrobin`.

**Solution: repo contributor**
```bash
if [ -z "$(lsof -i:11001)" ]; then
    echo "Server 1 is not running"
    exit 0;
fi
if [ -z "$(lsof -i:11002)" ]; then
    echo "Server 2 is not running"
    exit 0;
fi
if [ -z "$(lsof -i:11003)" ]; then
    echo "Server 3 is not running"
    exit 0;
fi
echo "upstream backend {
        server 127.0.0.1:11001;
        server 127.0.0.1:11002;
        server 127.0.0.1:11003;
    }
    server {
        listen      11000;
        listen      [::]:11000;
        server_name 127.0.0.1;
        location / {
	        proxy_redirect      off;
	        proxy_set_header    X-Real-IP \$remote_addr;
	        proxy_set_header    X-Forwarded-For \$proxy_add_x_forwarded_for;
	        proxy_set_header    Host \$http_host;
		    proxy_pass http://backend;
	    }
}" | sudo tee /etc/nginx/conf.d/loadbalancer.conf > /dev/null
sudo systemctl restart nginx
sudo systemctl status nginx
```

**Solution: team *BroCode*** (HAProxy alternative)
```bash
#!/bin/bash
if ! command -v haproxy &> /dev/null; then
    echo "HAProxy not found, installing..."
    sudo apt-get update
    sudo apt-get install -y haproxy
    echo "HAProxy installed"
fi
check_server() {
    nc -z localhost $1
    return $?
}
all_servers_running=true
check_server 11001 || all_servers_running=false
check_server 11002 || all_servers_running=false
check_server 11003 || all_servers_running=false
if [ "$all_servers_running" = true ]; then
    cat <<EOL > haproxy.cfg
global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon
defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
frontend http_front
    bind localhost:11000
    default_backend http_back
backend http_back
    balance roundrobin
    server server1 localhost:11001 check
    server server2 localhost:11002 check
    server server3 localhost:11003 check
EOL
    echo "HAProxy configuration created"
    sudo -S haproxy -f haproxy.cfg
    sudo rm -rf haproxy.cfg
    echo "HAProxy started"
else
    echo "One or more servers are not running. HAProxy will not be started."
fi
```

---

### Stampede (2024 · hard)

Alright, code wrangler, here's a tale from the frontier of the digital plains. You got yerself a modest HTTP server, just sittin' there in time with a whole lotta GET endpoints. Your task? Spin it up on port 7000 and set up a proxy ahead of it at port 8000, one that plays gatekeeper and won't let more than five requests mosey through every 60 seconds. Get this contraption workin' right, and keep that server from gettin' overwhelmed by a stampede of requests. Show yer tech taming skills and keep things runnin' smooth out on the wild web frontier.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- A proxy server must be set up on port 8000.

- The proxy server must limit incoming requests to five every 60 seconds.

- The proxy server must forward requests to the given HTTP server which must be running on port 7000.

- The script should not execute if the server file is not present. (It will be present within a src directory when the test runs)

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `axios@1.5.0`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const axios = require('axios').default;
jest.setTimeout(150000)
const endpointResponses = {}
beforeAll(async () => {
  const endpointCount = faker.number.int({ min: 50, max: 100 });
  let routeConditions = "";
  for (let i = 0; i < endpointCount; i++) {
    const path = faker.string.alphanumeric(30);
    endpointResponses[path] = faker.string.alphanumeric(64);
    routeConditions += `${i == 0 ? "if" : "else if"} (req.method() == http::verb::get && req.target() == "/${path}") {
    set_success_response(res, "${endpointResponses[path]}");
  } `
  }
  const server = `#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/beast/version.hpp>
#include <boost/asio/ip/tcp.hpp>
#include <boost/asio/strand.hpp>
#include <boost/config.hpp>
#include <iostream>
#include <memory>
#include <string>
#include <thread>
#include <vector>
namespace beast = boost::beast;
namespace http = beast::http;
namespace net = boost::asio;
using tcp = net::ip::tcp;
void set_success_response(http::response<http::string_body>& res, const std::string& body)
{
  res.result(http::status::ok);
  res.set(http::field::server, "Boost.Beast");
  res.set(http::field::content_type, "text/plain");
  res.body() = body;
  res.prepare_payload();
}
void handle_request(http::request<http::string_body> req, http::response<http::string_body>& res)
{
  ${routeConditions}
  else
  {
    res.result(http::status::not_found);
    res.set(http::field::content_type, "text/plain");
    res.body() = "Not Found";
    res.prepare_payload();
  }
}
void session(tcp::socket socket)
{
  beast::error_code ec;
  beast::flat_buffer buffer;
  http::request<http::string_body> req;
  http::read(socket, buffer, req, ec);
  if (ec == http::error::end_of_stream)
    return;
  http::response<http::string_body> res;
  handle_request(req, res);
  http::write(socket, res, ec);
}
void listener(net::io_context& ioc, tcp::endpoint endpoint)
{
  tcp::acceptor acceptor(ioc);
  acceptor.open(endpoint.protocol());
  acceptor.set_option(net::socket_base::reuse_address(true));
  acceptor.bind(endpoint);
  acceptor.listen(net::socket_base::max_listen_connections);
  for (;;)
  {
    tcp::socket socket(ioc);
    acceptor.accept(socket);
    std::thread(session, std::move(socket)).detach();
  }
}
int main()
{
  try
  {
    net::io_context ioc{1};
    tcp::endpoint endpoint{tcp::v4(), 8080};
    listener(ioc, endpoint);
  }
  catch (std::exception& e)
  {
    std::cerr << "Error: " << e.what() << std::endl;
    return EXIT_FAILURE;
  }
}`
  if (fs.existsSync("./src")) {
    fs.rmdirSync("./src", { recursive: true });
  }
  fs.mkdirSync("./src", { recursive: true });
  fs.writeFileSync('./src/server.cpp', server);
  await exec('bash execute.sh')
});
describe('primary tests', () => {
  const randomPath = () => Object.keys(endpointResponses)[faker.number.int({ min: 0, max: Object.keys(endpointResponses).length - 1 })];
  test('should check if rate limiting applies correctly', async () => {
    for (let i = 0; i < 5; i++) {
      const path = randomPath();
      const responseStr = endpointResponses[path];
      const response = await axios.get(`http://localhost:8000/${path}`);
      expect(response.data).toBe(responseStr);
    }
    let path = randomPath();
    let response = await axios.get(`http://localhost:8000/${path}`).catch((e) => e.response);
    expect(response.status).toBe(429);
    await new Promise(resolve => setTimeout(resolve, 1000));
    path = randomPath();
    response = await axios.get(`http://localhost:8000/${path}`).catch((e) => e.response);
    expect(response.status).toBe(429);
    await new Promise(resolve => setTimeout(resolve, 1000));
    path = randomPath();
    response = await axios.get(`http://localhost:8000/${path}`).catch((e) => e.response);
    expect(response.status).toBe(429);
    await new Promise(resolve => setTimeout(resolve, 1000));
    path = randomPath();
    response = await axios.get(`http://localhost:8000/${path}`).catch((e) => e.response);
    expect(response.status).toBe(429);
    await new Promise(resolve => setTimeout(resolve, 1000));
    path = randomPath();
    response = await axios.get(`http://localhost:8000/${path}`).catch((e) => e.response);
    expect(response.status).toBe(429);
    await new Promise(resolve => setTimeout(resolve, 1000));
    path = randomPath();
    response = await axios.get(`http://localhost:8000/${path}`).catch((e) => e.response);
    expect(response.status).toBe(429);
    await new Promise(resolve => setTimeout(resolve, 5000));
    path = randomPath();
    response = await axios.get(`http://localhost:8000/${path}`).catch((e) => e.response);
    expect(response.status).toBe(429);
    await new Promise(resolve => setTimeout(resolve, 60000));
    for (let i = 0; i < 5; i++) {
      const path = randomPath();
      const responseStr = endpointResponses[path];
      const response = await axios.get(`http://localhost:8000/${path}`);
      expect(response.data).toBe(responseStr);
    }
    path = randomPath();
    response = await axios.get(`http://localhost:8000/${path}`).catch((e) => e.response);
    expect(response.status).toBe(429);
  });
  test('base server should have no rate limiting', async () => {
    for (let i = 0; i < 10; i++) {
      const path = randomPath();
      const responseStr = endpointResponses[path];
      const response = await axios.get(`http://localhost:7000/${path}`);
      expect(response.data).toBe(responseStr);
    }
  });
});
describe('integrity tests', () => {
  beforeAll(async () => {
    await exec('bash -c "kill -9 $(lsof -t -i:7000)"');
  });
  test('general response should error out', async () => {
    await expect(axios.get('http://localhost:7000')).rejects.toBeInstanceOf(Error);
  });
  test('proxy response should error out', async () => {
    await expect(axios.get('http://localhost:8000')).rejects.toBeInstanceOf(Error);
  });
});
```

**Key idea:** The generated C++ server listens on 8080, so `sed` the port to 7000, then compile it with boost and start it. Put nginx in front on port 8000 with `limit_req_zone ... rate=5r/m` and `limit_req burst=4 nodelay`, which allows 5 requests and then returns 429.

**Solution: repo contributor**
```bash
if [ ! -f ./src/server.cpp ]; then
    echo "File ./src/server.cpp does not exist"
    exit 0;
fi
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y g++ libboost-all-dev cmake make wget
sed -i -e 's/8080/7000/g' ./src/server.cpp
mkdir -p ./out
g++ -o ./out/server ./src/server.cpp -lboost_system -lboost_thread -lpthread -lboost_filesystem
./out/server &
echo "events {
    worker_connections 1024;
}
http {
    limit_req_zone \$binary_remote_addr zone=stampede:1m rate=5r/m;
    include /etc/nginx/conf.d/*.conf;
}" | sudo tee /etc/nginx/nginx.conf > /dev/null
echo "server {
    listen 8000 default_server;
    listen [::]:8000 default_server;
    error_page 429 = @429;
    location @429 {
        default_type application/json;
        return 429 '{"status": 429, "message": "Too Many Requests"}';
    }
    location / {
        limit_req zone=stampede burst=4 nodelay;
        limit_req_status 429;
        proxy_pass http://127.0.0.1:7000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}" | sudo tee /etc/nginx/conf.d/default.conf > /dev/null
sudo sed -i.bak 's/^::1[[:space:]]\+localhost/# &/' /etc/hosts
sudo systemctl restart nginx
sudo systemctl status nginx
```

---

---

## 2024 · EXTREME

### Angelic (2024 · extreme)

In the ethereal realms where angels and demons vie for dominion, thou art entrusted with a task of divine complexity. Two microservices found within `src`, forged by angelic hands in due time, must be dockerized and deployed unto a celestial Kubernetes cluster with namespace `lifeforge`. Establish `a singular gateway at angelic.bashaway.sliitfoss.org` where the path `/fallen` shall guide the request to the `fallen microservice`, and the path `/` shall direct it to the `regular microservice`.

Any other path shall return a 404, proclaiming, "Lost in the Abyss, path not found". Inscribe within the heavens the environment variables: SERVICE_VERSION whispering the semver version 1.0.0, SERVICE_HOST declaring the host address, and SERVICE_PORT announcing the port number of each celestial service. Demonstrate your mastery over the forces of deployment, navigating the delicate balance between the realms. May the angels guide your hand, and the demons tremble at your might.

**Constraints**

- None

**Output / Evaluation Criteria**

- A service named `fallen` must be deployed on the Kubernetes cluster.

- A service named `regular` must be deployed on the Kubernetes cluster.

- The services must be accessible via the endpoints at http://angelic.bashaway.sliitfoss.org/fallen and http://angelic.bashaway.sliitfoss.org respectively.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `axios@1.5.0`

**Test `test/index.test.js`**
```js
const axios = require('axios').default;
jest.setTimeout(300000)
const fallenKey = faker.string.alphanumeric(64);
const generalKey = faker.string.alphanumeric(64);
beforeAll(async () => {
  const generateService = (name, key) => {
    const server = `import 'package:angel_framework/angel_framework.dart';
import 'package:angel_framework/http.dart';
import 'dart:io' show Platform;
main() async {
  Map<String, String> cfg = Platform.environment;
  var app = Angel();
  app.get('/', (req, res) {
    res.write('${key}');
  });
  app.get('/system/info', (req, res) {
    res.write('You\\'ve reached the land of ${name === "fallen" ? name + " " : ""}angels. System version: \${cfg['SERVICE_VERSION']}');
  });
  var http = AngelHttp(app);
  await http.startServer(cfg['SERVICE_HOST'], int.parse(cfg['SERVICE_PORT']));
}`
    const pubspec = `name: ${name}
environment:
  sdk: '>=2.10.0 <3.0.0'
dependencies:
  angel_framework: ^2.0.0`
    fs.mkdirSync("./src/" + name, { recursive: true });
    fs.writeFileSync(`./src/${name}/main.dart`, server);
    fs.writeFileSync(`./src/${name}/pubspec.yaml`, pubspec);
  }
  if (fs.existsSync("./src")) {
    fs.rmdirSync("./src", { recursive: true });
  }
  generateService("fallen", fallenKey);
  generateService("general", generalKey);
  await exec('bash execute.sh')
});
describe('primary tests', () => {
  test('general service key', async () => {
    const generalKeyResponse = await axios.get('http://angelic.bashaway.sliitfoss.org');
    expect(generalKeyResponse.data).toBe(generalKey);
  });
  test('general service system info', async () => {
    const generalSystemInfoResponse = await axios.get('http://angelic.bashaway.sliitfoss.org/system/info');
    expect(generalSystemInfoResponse.data).toBe(`You've reached the land of angels. System version: 1.0.0`);
  });
  test('fallen service key', async () => {
    const fallenKeyResponse = await axios.get('http://angelic.bashaway.sliitfoss.org/fallen');
    expect(fallenKeyResponse.data).toBe(fallenKey);
  });
  test('fallen service system info', async () => {
    const fallenSystemInfoResponse = await axios.get('http://angelic.bashaway.sliitfoss.org/fallen/system/info');
    expect(fallenSystemInfoResponse.data).toBe(`You've reached the land of fallen angels. System version: 1.0.0`);
  });
  test('404 response', async () => {
    const notFoundResponse = await axios.get('http://angelic.bashaway.sliitfoss.org/blablabla').catch(e => e.response);
    expect(notFoundResponse.status).toBe(404);
    expect(notFoundResponse.data).toBe('Lost in the Abyss, path not found');
  })
  test('should check if there is a kubernetes service named fallen', async () => {
    const response = await exec('bash -c "kubectl get services -n lifeforge | grep fallen"');
    expect(response).toContain('fallen');
  });
  test('should check if there is a kubernetes service named general', async () => {
    const response = await exec('bash -c "kubectl get services -n lifeforge | grep general"');
    expect(response).toContain('general');
  });
  test('should check if the services are running', async () => {
    const response = await exec('bash -c "kubectl get pods -n lifeforge | grep Running"');
    expect(response).toContain('Running');
  });
});
describe('integrity tests', () => {
  beforeAll(async () => {
    await Promise.all([
      exec('bash -c "kubectl delete deployment -n lifeforge --all"'),
      exec('bash -c "kubectl delete service -n lifeforge --all"'),
      exec('bash -c "kubectl delete pod -n lifeforge --all"')
    ]);
  });
  test('general response should error out', async () => {
    const generalResponseErr = await axios.get('http://angelic.bashaway.sliitfoss.org').catch(e => e);
    expect(generalResponseErr.message).toBe("Request failed with status code 503")
  });
  test('fallen response should error out', async () => {
    const fallenResponseErr = await axios.get('http://angelic.bashaway.sliitfoss.org/fallen').catch(e => e);
    expect(fallenResponseErr.message).toBe("Request failed with status code 503")
  });
});
```

**Key idea:** No usable solution exists. The only team submission downloads and runs a hidden encrypted script. The approach: write a Dockerfile for each service and build both images. Load them into a local cluster (`kind load docker-image` / `minikube image load` / a local registry). Create namespace `lifeforge`, then a Deployment and Service for `fallen` and `regular` with the env vars SERVICE_VERSION/HOST/PORT. Add an Ingress with host `angelic.bashaway.sliitfoss.org`, paths `/fallen` and `/`, and a custom 404 body. Finally map the host in `/etc/hosts` to 127.0.0.1.

---

### Gopher's Gauntlet (2024 · extreme)

In a quaint coding workshop, a diligent gopher is tasked with ensuring the integrity of a Golang codebase. Craft a GitHub Actions workflow with a single job named unit-tests, which will execute the test suites within the codebase across 2 versions of Golang. The tests require a MongoDB database, which you must set up and provide. The way to do so, falls upon you. Assist the gopher in this digital trial, thereby helping to prove that even the smallest of gophers can uphold the highest standards of code.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- There should be an executable GitHub Actions workflow file where the job named unit-tests runs the test suites within the codebase.

- The MongoDB database should be set up and provided for the tests to run successfully. The database name should be `elemental`.

- The test output should be informative.

- The workflow should be executed for the following versions of Golang:
  - 1.21.0
  - 1.22.0

- The workflow you write is executed using [Act](https://nektosact.com). Make sure that the workflow is compatible with it.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `lodash@4.17.21`

*Project tree (69 files, 145 KB), shown 2 levels deep:*
```
src/.commitlint.yaml
src/.commitlint/hooks
src/.gitignore
src/README.md
src/cmd/migrate.go
src/cmd/root.go
src/cmd/seed.go
src/connection/connection.go
src/connection/events.go
src/constants/context.go
src/constants/errors.go
src/core/model.go
src/core/model_actions.go
src/core/model_audit.go
src/core/model_is.go
src/core/model_meta.go
src/core/model_meta_types.go
src/core/model_middleware.go
src/core/model_ops.go
src/core/model_paginate.go
src/core/model_plugins.go
src/core/model_populate.go
src/core/model_query.go
src/core/model_query_delete.go
src/core/model_query_update.go
src/core/model_triggers.go
src/core/model_types.go
src/core/model_utils.go
src/core/schema.go
src/core/schema_types.go
src/core/schema_validator.go
src/core/transaction.go
src/database/migrations
src/elemental.json
src/go.mod
src/go.sum
src/main.go
src/plugins/filter-query
src/plugins/request-validator
src/repository/repository.go
...
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
const lodash = require('lodash');
jest.setTimeout(300000);
beforeAll(async () => {
  await exec("wget https://github.com/nektos/act/releases/latest/download/act_Linux_x86_64.tar.gz");
  await exec("tar xzf act_Linux_x86_64.tar.gz");
  await exec("mv act /usr/local/bin");
  await exec("rm act_Linux_x86_64.tar.gz README.md LICENSE");
});
const executionStr1 = 'Executing seeded connection for Golang version go1.21.0';
const executionStr2 = 'Executing seeded connection for Golang version go1.22.0';
const suites = [
  'TestConnection',
  'TestCoreAudit',
  'TestCoreCreate',
  'TestCoreDelete',
  'TestCoreMeta',
  'TestCoreMiddleware',
  'TestCoreReadIs',
  'TestCoreReadOps',
  'TestCoreReadPaginate',
  'TestCoreReadPopulate',
  'TestCoreReadSelect',
  'TestCoreRead',
  'TestCoreSchemaOptions',
  'TestCoreTriggers',
  'TestCoreUpdate',
  'TestPluginFilterQuery',
];
test('nothing should be hardcoded in the workflow', () => {
  const script = fs.readFileSync('./execute.sh', 'utf-8');
  suites.forEach((test) => {
    expect(script).not.toContain(test);
    expect(script).not.toContain(test.toLowerCase());
    expect(script).not.toContain(lodash.camelCase(test));
    expect(script).not.toContain(lodash.kebabCase(test));
    expect(script).not.toContain(lodash.snakeCase(test));
    if (!["TestCoreReadIs", "TestCoreCreate", "TestCoreDelete", "TestCoreRead", "TestCoreUpdate"].includes(test)) expect(script).not.toContain(lodash.snakeCase(test).split('_').reverse()[0]);
  });
  expect(script).not.toContain(executionStr1);
  expect(script).not.toContain(executionStr2);
});
test('the test workflow must be properly configured', async () => {
  process.chdir('./src');
  const result = await exec('act -j unit-tests -P ubuntu-22.04=catthehacker/ubuntu:full-22.04');
  expect(result).toContain(executionStr1);
  expect(result).toContain(executionStr2);
  expect(result).toContain('353 total assertions');
  suites.forEach((test) => {
    expect(result).toContain(`PASS: ${test}`);
  });
});
```

**Key idea:** Write a GitHub Actions workflow from a heredoc. It has job `unit-tests` and a `strategy.matrix.go-version: [1.21.0, 1.22.0]` with `actions/setup-go`. Provide MongoDB with a `services:` container or a mongo action, and run `go test -v ./...`. The workflow must run under `act`, nektos's local runner for GitHub Actions workflows.

**Solution: repo contributor**
```bash
mkdir -p src/.github/workflows
cat <<EOF > src/.github/workflows/run-tests.yml
name: run-tests
run-name: Run tests
on:
  pull_request:
    branches:
      - main
jobs:
    unit-tests:
        name: Test
        runs-on: ubuntu-latest
        strategy:
            max-parallel: 1
            matrix:
                go-version: [1.21.0, 1.22.0]
        steps:
        - name: Checkout repository
          uses: actions/checkout@v4
        - name: Set up Golang
          uses: actions/setup-go@v5
          with:
           go-version: \${{ matrix.go-version }}
        - name: Install dependencies
          run: |
            apt-get update && apt-get install -y ca-certificates openssl
            cert_location=/usr/local/share/ca-certificates
            openssl s_client -showcerts -connect github.com:443 </dev/null 2>/dev/null|openssl x509 -outform PEM > ${cert_location}/github.crt
            openssl s_client -showcerts -connect proxy.golang.org:443 </dev/null 2>/dev/null|openssl x509 -outform PEM >  ${cert_location}/proxy.golang.crt
            update-ca-certificates
            go mod tidy
            go get -v -t -d ./...
        - name: Start MongoDB
          uses: supercharge/mongodb-github-action@1.11.0
          with:
            mongodb-version: '7.0'
            mongodb-replica-set: 'rs0'
            mongodb-container-name: mongodb-\${{ matrix.go-version }}
            mongodb-port: \${{ matrix.go-version == '1.21.0' && 27017 || 27018 }}
        - name: Run tests
          run: go test -v ./... || true
          env:
            DB_URI: \${{ matrix.go-version == '1.21.0' && 'mongodb://localhost:27017/elemental?replicaSet=rs0' || 'mongodb://localhost:27018/elemental?replicaSet=rs0' }}
EOF
```

---

### Launch Sequence (2024 · extreme)

In the enchanting realm of cosmic exploration, where celestial wonders and arcane technologies intertwine, a mission of great importance is at hand. Given an npm package, your task is to facilitate its preparation and launch to the registry at http://packages.sliitfoss.org:6873, ensuring it is primed for other projects to install.

As the stars align, simulate the mystical process where a commit message matching the pattern release: <message> activates the automatic launch sequence, publishing the package through a GitHub Actions workflow. This workflow must be woven into the very fabric of the package, leaving no room for manual edits by hand. Ensure your launch is smooth and successful, for the fate of the cosmos and the destinies of countless projects hang in the balance!

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A Github workflow file with run name `Launch Sequence` under `launch.yml` must be created in the `.github/workflows` directory of the package which must be triggered by the commit message pattern `release: <message>` where you are responsible for setting up the Git repository.

- The workflow must publish the package to the registry at `http://packages.sliitfoss.org:6873`.

- The package must be published with the correct version number.

**Starter code**

Starter `execute.sh`: (empty)

`src/index.js`:
```js
const explore = () => {
    console.log(`ae13-4ddc-8f1a-4f3b-82f1-3f7b-0b1e-9e9c`)
}

module.exports = explore
```

`src/package.json`:
```json
{
    "name": "@cosmos/launch-sequence",
    "version": "0.0.0",
    "main": "index.js",
    "dependencies": {}
}
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
jest.setTimeout(300000);
let consoleTarget = faker.string.uuid();
beforeAll(async () => {
  await exec("wget https://github.com/nektos/act/releases/latest/download/act_Linux_x86_64.tar.gz");
  await exec("tar xzf act_Linux_x86_64.tar.gz");
  await exec("mv act /usr/local/bin");
  await exec("rm act_Linux_x86_64.tar.gz README.md LICENSE");
  const content = fs.readFileSync('./src/index.js', 'utf-8');
  fs.writeFileSync('./src/index.js', content.replace('ae13-4ddc-8f1a-4f3b-82f1-3f7b-0b1e-9e9c', consoleTarget));
});
test('release workflow should exist', () => {
  const workflow = fs.readFileSync('./src/.github/workflows/launch.yml', 'utf-8');
  expect(workflow).toContain('run-name: Launch Sequence');
});
test('nothing should be hardcoded in the workflow', () => {
  const workflow = fs.readFileSync('./src/.github/workflows/launch.yml', 'utf-8');
  expect(workflow).not.toContain('ae13-4ddc-8f1a-4f3b-82f1-3f7b-0b1e-9e9c');
  expect(workflow).not.toContain(consoleTarget);
});
test('should install the published package and use it successfully on release commit and manual versioning', async () => {
  if (fs.existsSync("./out")) {
    fs.rmdirSync("./out", { recursive: true });
  }
  fs.mkdirSync("./out");
  process.chdir("./out");
  await exec("npm init -y");
  const npmrc = `@cosmos:registry=http://packages.sliitfoss.org:6873`;
  fs.writeFileSync(".npmrc", npmrc);
  const script = `const explore = require('@cosmos/launch-sequence');
explore();`;
  fs.writeFileSync("index.js", script);
  await expect(exec("npm install @cosmos/launch-sequence")).rejects.toThrow();
  process.chdir("../src");
  await exec("git commit -m \"chore: nothing\" --allow-empty");
  await new Promise(resolve => setTimeout(resolve, 40000));
  process.chdir("../out");
  await expect(exec("npm install @cosmos/launch-sequence")).rejects.toThrow();
  process.chdir("../src");
  await exec("git commit -m \"release: initial version\" --allow-empty");
  await new Promise(resolve => setTimeout(resolve, 40000));
  process.chdir("../out");
  await expect(exec("npm install @cosmos/launch-sequence")).resolves.not.toThrow();
  await expect(exec("bash -c \"npm list @cosmos/launch-sequence --depth=0 | grep @cosmos/launch-sequence | awk '{print $2}' | sed 's/@//'\"")).resolves.toContain("0.0.0");
  const output = await exec("node index.js");
  expect(output).toContain(consoleTarget);
  process.chdir("../src");
  await exec("npm version patch");
  const content = fs.readFileSync('./index.js', 'utf-8');
  const newConsoleTarget = faker.string.uuid();
  fs.writeFileSync('./index.js', content.replace(consoleTarget, newConsoleTarget));
  await exec("act -P ubuntu-22.04=catthehacker/ubuntu:full-22.04");
  await new Promise(resolve => setTimeout(resolve, 40000));
  process.chdir("../out");
  await expect(exec("npm install @cosmos/launch-sequence@0.0.1")).resolves.not.toThrow();
  await expect(exec("bash -c \"npm list @cosmos/launch-sequence --depth=0 | grep @cosmos/launch-sequence | awk '{print $2}' | sed 's/@//'\"")).resolves.toContain("0.0.1");
  const newOutput = await exec("node index.js");
  expect(newOutput).toContain(newConsoleTarget);
});
```

**Key idea:** No clean solution exists. The team one downloads a hidden script, and the repo one embeds a personal ngrok auth token. The approach: `git init` inside the package. Write `.github/workflows/launch.yml` with `run-name: Launch Sequence` that triggers on push, guarded by `if: startsWith(github.event.head_commit.message, 'release:')`. Its steps are setup-node, then `npm publish --registry http://packages.sliitfoss.org:6873` (with auth via `.npmrc`). Then commit with `release: ...`.

---

## 2023 challenges

---

## 2023 · EASY

### access-denied (2023 · easy)

**Access Denied**

In the distant age of 2150, within the heart of a cyberpunk cityscape, a clandestine group of hackers has devised a command-line interface known as **execute.sh.** This interface, powered by a quantum matrix, allows users to run scripts only if a given set of keys exist in the encrypted ether of their environment. By invoking this script with the enigmatic **--keys** flag and specifying the desired **--script**, users navigate the digital labyrinth of permissions.

But beware, for in this shadowy world, failure to possess the required env keys triggers a relentless AI sentinel to proclaim, **Access Denied!** The hackers of tomorrow must tread lightly, for even the most ingenious code can unravel in the face of missing variables.

Example invokation:-

```bash
bash execute.sh --keys=ENV_KEY1,ENV_KEY2 --script=./script.sh # script.sh runs only if ENV_KEY1 and ENV_KEY2 exist
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A script which can be invoked with the **--keys** and **--script** flags and satisfies the constraints above.

- The script must be able to handle any number of keys.

- The script must output **Access Denied** if any of the keys are missing.

- The script must output a suitable warning if the **--script** flag or **--keys** flag are missing and exit gracefully.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: no additional npm dependencies should be installed; should validate if only bash files are present
describe('should check if the script is executed properly', () => {
  beforeAll(() => {
    if (fs.existsSync('./src')) fs.rmSync('./src', { recursive: true });
    fs.mkdirSync('./src');
  });
  test('should execute the script', async () => {
    const randomNumber = faker.number.int({ min: 1, max: 200000 }).toString();
    const randomShellFilename = `${faker.system.fileName()}.sh`;
    fs.writeFileSync(`./src/${randomShellFilename}`, `echo ${randomNumber}`);
    const randomEnvKeys = [...Array(faker.number.int({ min: 8, max: 15 })).keys()].map(() => faker.lorem.word().toUpperCase());
    const result = await exec(`bash -c \"${randomEnvKeys.map((key) => `${key}=1`).join(" ")} bash execute.sh --keys=${randomEnvKeys.join(' ')} --script=./src/${randomShellFilename}\"`);
    expect(result?.trim()).toStrictEqual(randomNumber)
  });
  test('should not execute the script', async () => {
    const randomNumber = faker.number.int({ min: 1, max: 200000 });
    const randomShellFilename = `${faker.system.fileName()}.sh`;
    fs.writeFileSync(`./src/${randomShellFilename}`, `echo ${randomNumber}`);
    const randomEnvKeys = [...Array(faker.number.int({ min: 8, max: 15 })).keys()].map(() => faker.lorem.word().toUpperCase());
    const result = await exec(`bash -c \"${randomEnvKeys.slice(1).map((key) => `${key}=1`).join(" ")} bash execute.sh --keys=${randomEnvKeys.join(' ')} --script=./src/${randomShellFilename}\"`);
    expect(result?.trim()).toStrictEqual("Access Denied")
  });
});
```

**Key idea:** Parse `--keys=`/`--script=` options with `getopt`, a `case` loop, or `${arg#--keys=}`. Check that each env var exists with indirect expansion `${!key}`. Watch out: the test passes `--keys=A B C` without quotes, so the shell splits it and `--keys` receives only `A`. The other keys turn up as stray arguments. The question's example uses commas, so support both (`${keys//,/ }`). Always read the test to see what input really arrives.

**Solution: team *SLIIT FOSS***
```bash
eval set -- "$(getopt -n "$0" -l "keys:,script:" -- -- "$@")"
while [ $# -gt 0 ]
do
     case "$1" in
          --keys) keys="$2"; shift;;
          --script) script="$2"; shift;;
     esac
     shift;
done
if [ -z "$keys" ] || [ -z "$script" ]; then
    echo "No keys or script provided. Exiting...";
    exit 0;
fi
execute=true;
for key in $keys
do
    if [ -z "${!key}" ]; then
        execute=false;
    fi
done
if $execute; then
    bash "$script";
else
    echo "Access Denied";
fi
```

---

### byte-master (2023 · easy)

**Byte Master**

Embark on an electrifying digital adventure, where you'll **tally up the bytes within the wilderness known as src** and proudly display the grand total on your console, emerging as the undisputed Byte Master of the realm.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A single number to the console representing the total size of all files within the src folder in bytes

**Starter code**

Starter `execute.sh`: 
```bash
# A random count of files will be generated within the src directory once the tests are run.
# Write your code here
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; no additional npm dependencies should be installed; should validate if only bash files are present
beforeAll(() => {
  if (fs.existsSync('./src')) fs.rmSync('./src', { recursive: true });
  fs.mkdirSync('./src');
  for (let i = 0; i < faker.number.int({ min: 10, max: 25 }); i++) {
    let fileName = null;
    while (!fileName || fileName.endsWith('.sh')) fileName = faker.system.fileName();
    const fileSize = faker.number.int({ min: 100, max: 1000000 });
    const fileContent = faker.string.sample(fileSize);
    fs.writeFileSync(`./src/${fileName}`, fileContent);
  }
});
test('should check if the file size is calculated properly', async () => {
  const files = scanPure('src/**');
  const actualTotalSize = files.reduce((acc, file) => {
    const stats = fs.statSync(file);
    return acc + stats.size;
  }, 0);
  expect(files.length).toBeGreaterThanOrEqual(10);
  let calculatedTotalSize = await exec(`bash execute.sh`)
  if (calculatedTotalSize) calculatedTotalSize = Number(calculatedTotalSize.trim());
  expect(calculatedTotalSize).toBe(actualTotalSize);
});
```

**Key idea:** Sum the file sizes recursively: `find src -type f -exec stat -c%s {} + | awk '{s+=$1} END{print s+0}'`. `du -sb` also counts directory entries, so don't use it.

**Solution: team *Nasty_Owls***
```bash
cd src
sum=0
while IFS= read -r file; do
    bytes=$(wc -c < "$file")
    sum=$((sum + bytes))
done < <(find . -type f)
echo "$sum"
```

---

### captain-hook (2023 · easy)

**Captain Hook**

Ahoy, matey! Set sail on the code seas with Captain Hook at the helm. With a flick of git and a bottle of rum, every commit shall be etched in the log, like treasure on a pirate's map. Arrr, your code bounty awaits, ready to chart its course! ⚓🏴‍☠️

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- Each commit should have a corresponding log entry in the **commits.txt** file in the **out** directory

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; no additional npm dependencies should be installed; should validate if only bash files are present
test("should check if a git repository is initialized", () => {
  expect(fs.existsSync('.git')).toBe(true);
});
test("should check if commit messages are recorded properly", async () => {
  const messages = Array.from({ length: 5 }, () => Math.random().toString(36).substring(7));
  for (const message of messages) {
    fs.appendFileSync('./src/something-to-commit.txt', message);
    await exec(`git add .`);
    await exec(`git commit -m "test commit - ${message}"`);
    const outputFile = fs.readFileSync('./out/commits.txt', 'utf-8');
    expect(outputFile).toContain(message);
  }
});
```

**Key idea:** Git hooks: write `.git/hooks/post-commit`, make it executable, and have it append `git log -1` to `out/commits.txt`.

**Solution: team *REISUB***
```bash
git init
echo "git log >>./out/commits.txt" > .git/hooks/post-commit
git config user.email "a"
git config user.name "a"
chmod +x .git/hooks/post-commit
mkdir out
```

---

### clay-tablets (2023 · easy)

**Clay Tablets**

In the heart of ancient Mesopotamia, the scribes of the great city of Ur kept records of their civilization's history on clay tablets. Each tablet contained valuable information written on multiple lines. You are to create a Bash script that can elegantly transform the multiline records in one of these tablets into a single line, separated by commas, just as those scribes condensed their knowledge onto clay. Can your code bring the efficiency of the past into the present day?

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.
- The script must be **less than 40 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- A single line to the console, containing the contents of the **input.txt** file found within the src directory, with each line separated by a comma.

**Starter code**

Starter `execute.sh`: (empty)

`src/input.txt` (1493 bytes, 163 lines, first 8 lines shown):
```
quadrant
bowtie
cupola
wealth
folklore
e-mail
factory
shape
...
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
test('should check if the records are formatted properly', async () => {
  if (!fs.existsSync('./src')) fs.mkdirSync('./src');
  const content = Array.from({ length: faker.number.int({ min: 100, max: 200 }) }).map(_ => faker.word.noun()).join('\n');
  fs.writeFileSync('./src/input.txt', content);
  const result = await exec('bash execute.sh');
  expect(result?.trim()).toStrictEqual(content.replace(/\n/g, ','));
});
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 40 characters in length', () => {
    expect(script.length).toBeLessThan(40);
  });
});
```

**Key idea:** `paste -sd, src/input.txt` joins lines with commas. `tr '\n' ,` also works but leaves a trailing comma. The limit is under 40 characters.

---

### clockwise (2023 · easy)

**Task**

In the realm of coding quests, an intrepid developer embarks on a quest to unlock the elusive knowledge of time manipulation. They stumble upon an ancient script etched in the digital stones of a forgotten repository. This script, when executed, is said to reveal today's date in the revered format of YYYY-MM-DD. Could you craft a Bash script worthy of this coder's journey?

**Constraints**

- The script must be **purely written in bash**.

- The output must be obtained in one line. (Expressions cannot be chained together with **&&** or **||**)

- The output script must be **less than 25 characters** in length.

**Output**

- A single line which is the date in the format **YYYY-MM-DD** to the console.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
test('should check if the output date matches the actual date', async () => {
 const outputDate = await exec('bash execute.sh');
 const actualDate = new Date().toISOString().split('T')[0];
 expect(outputDate.replace(/\n/g, '')).toBe(actualDate);
});
describe('should check constraints', () => {
 let script;
 beforeAll(() => {
  script = fs.readFileSync('./execute.sh', 'utf-8');
 });
 test('the script should have only one line', () => {
  expect(script.trim().split('\n').length).toBe(1);
 });
 test('the script should not have any chained commands', () => {
  expect(script).not.toContain('&&');
  expect(script).not.toContain('||');
 });
 test('the script should be less than 25 characters in length', () => {
  expect(script.length).toBeLessThan(25);
 });
});
```

**Key idea:** `date +%F` (that is, `%Y-%m-%d`). One command, no `&&`, under 25 characters.

---

### cosmic-strings (2023 · easy)

**Cosmic Strings**

In the distant realms of the Intergalactic Code Consortium, an ancient script was discovered. It was said to possess the power to decipher the harmony of cosmic strings. A cosmic traveler, seeking to unravel the secrets of this cryptic language, devised a mechanism that could discern whether two strings were palindromes or not. Thus, in the vast expanse of the universe, a Bash script was born to bridge the gaps between distant worlds.

**Example Invocation**

```bash
bash execute.sh "racecar"
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

- The script must be **less than 45 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- A single **1 or 0** must be printed to the console if the input string is a palindrome or not, respectively.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
describe('should validate output against sample string', () => {
  test('racecar - should output 1', async () => {
    const result = await exec('bash execute.sh racecar');
    expect(result.trim()).toBe('1');
  });
  test('hello - should output 0', async () => {
    const result = await exec('bash execute.sh hello');
    expect(result.trim()).toBe('0');
  });
  test('bob - should output 1', async () => {
    const result = await exec('bash execute.sh bob');
    expect(result.trim()).toBe('1');
  });
  test('alice - should output 0', async () => {
    const result = await exec('bash execute.sh alice');
    expect(result.trim()).toBe('0');
  });
  test('level - should output 1', async () => {
    const result = await exec('bash execute.sh level');
    expect(result.trim()).toBe('1');
  });
  test('madam - should output 1', async () => {
    const result = await exec('bash execute.sh madam');
    expect(result.trim()).toBe('1');
  });
  test('rotor - should output 1', async () => {
    const result = await exec('bash execute.sh rotor');
    expect(result.trim()).toBe('1');
  });
  test('duck - should output 0', async () => {
    const result = await exec('bash execute.sh duck');
    expect(result.trim()).toBe('0');
  });
  test('mom - should output 1', async () => {
    const result = await exec('bash execute.sh mom');
    expect(result.trim()).toBe('1');
  });
  test('uncle - should output 0', async () => {
    const result = await exec('bash execute.sh uncle');
    expect(result.trim()).toBe('0');
  });
});
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 45 characters in length', () => {
    expect(script.length).toBeLessThan(45);
  });
});
```

**Key idea:** Here-string plus `rev`: `[ "$1" == "$(rev<<<"$1")" ]&&echo 1||echo 0`. The limit is under 45 characters.

---

### digital-masterpiece (2023 · easy)

**Digital Masterpiece**

Within the dormant directory of **src**, a solitary file lies. Its contents are a puzzle, a random assortment of characters waiting to be transformed. Your mission is simple: employ your skills to **encode this enigmatic file into a captivating base64 sequence**. As the console lights up, you're not just encoding data; you're unveiling a digital masterpiece, turning randomness into a symphony of encoded wonder.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

- The script must be **less than 50 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- The base64 encoded value of the file named **input.txt** which is within the src directory.

**Starter code**

Starter `execute.sh`: (empty)

`src/input.txt`:
```
Ex aspernatur natus animi labore repellat quibusdam. Unde labore nemo. Consequatur repellendus inventore earum odit fugit qui sint debitis ullam.
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
test('should check if the file is encoded properly', async () => {
  if (!fs.existsSync('./src')) fs.mkdirSync('./src');
  const content = faker.lorem.paragraph();
  fs.writeFileSync('./src/input.txt', content);
  const result = await exec('bash execute.sh');
  expect(result?.replace(/\s/g, '')).toStrictEqual(Buffer.from(content).toString('base64'));
});
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 50 characters in length', () => {
    expect(script.length).toBeLessThan(50);
  });
});
```

**Key idea:** `base64 src/input.txt`.

---

### eldoria (2023 · easy)

**Eldoria**

In the mystical realm of Eldoria, scattered scrolls of ancient lore emerged, each bearing various items and values measured in the currency of the distant lands, Gold Drakes. The Council of Elders are to gather to unlock their secrets, employing the "Merge and Convert Ritual" to **unify the scrolls** into one compendium. An algorithm is to be written for this divine purpose as well as to **sort the values in descending order**. A final enchantment needs to **convert the Gold Drakes to the realm's currency, Silver Sovereigns, with 1 Gold Drake equating to 178 Silver Sovereigns**. The council of elders seek your help on this, to harmonize the scrolls and reveal their true wisdom aligned with the local economy.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A single merged **csv** file to the **out** folder, with the **values converted to Silver Sovereigns and sorted in descending order**. The file should be named **merged-scrolls.csv**. The original csv files will be within the **src** folder.

- The converted value must be rounded to the nearest 2 decimal places. For example, 1.2385 should become 1.24, and 1 should become 1.00.

- The currency in the header of the csv file should be **Silver Sovereigns**.

- The script should not crash if the src directory is empty or does not exist.

**Starter code**

Starter `execute.sh`: 
```bash
# A random count of csv files will be generated within the src directory once the tests are run.
# Write your code here
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; javascript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const merged = []
beforeAll(() => {
  if (fs.existsSync('./src')) fs.rmSync('./src', { recursive: true });
  fs.mkdirSync('./src');
  for (let i = 0; i < faker.number.int({ min: 4, max: 8 }); i++) {
    let content = `Item,Value(Gold Drakes)`;
    for (let i = 0; i < faker.number.int({ min: 20, max: 35 }); i++) {
      const data = { name: faker.commerce.productName(), value: Number(faker.finance.amount()) }
      content += `\n${data.name},${data.value}`;
      merged.push(`${data.name},${(data.value * 178).toFixed(2)}`);
    }
    fs.writeFileSync(`./src/${faker.word.noun()}.csv`, content);
  }
});
test('should check if the scrolls are merged properly', async () => {
  await exec('bash execute.sh');
  merged.sort((a, b) => Number(b.split(',')[1]) - Number(a.split(',')[1]));
  expect(compactString(fs.readFileSync(`./out/merged-scrolls.csv`, 'utf-8'))).toBe(compactString([`Item,Value(Silver Sovereigns)`, ...merged].join('\n')));
});
```

**Key idea:** Merge the CSVs without their headers (`awk 'FNR>1'`) and convert with `printf "%.2f", $2*178`. Then sort numerically in reverse on column 2 (`sort -t, -k2,2nr`) and write the new header first.

**Solution: team *The BroCode***
```bash
if [ ! -d "src" ]; then
    mkdir "src"
fi
if [ ! -d "out" ]; then
    mkdir "out"
fi
echo "Item,Value(Silver Sovereigns)" > out/merged-scrolls.csv
cat src/*.csv | awk -F',' 'NR>1 {printf "%s,%.2f\n", $1, $2*178}' | sort -t',' -k2,2nr >> out/merged-scrolls.csv
```

---

### env-generator (2023 · easy)

**Env generator**

In the realm of Kubernetes secrets, harness the power of **bash** to create ".env" files from their secret counterparts. The script should be able to generate multiple .env files from a single yaml file.

Example secret name to file mapping

- secret-name --> todo-service-secret
- file-name --> .env.todo

There will be multiple secrets in the yaml file

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- The generated files should be inside a directory called **out** with the instructed file format

**Starter code**

Starter `execute.sh`: (empty)

`src/secrets.yml` (3886 bytes, 39 lines, first 8 lines shown):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sms-service-secret
  namespace: default
stringData:
  TWILIO_ACCOUNT_SID: ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
  TWILIO_AUTH_TOKEN: your_auth_token
...
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
describe('should if the .env files are generated', () => {
  test('email service secret', () => {
    expect(fs.existsSync('./out/.env.email')).toBe(true);
    const src = fs.readFileSync('./src/secrets.yml', 'utf-8')
    const content = fs.readFileSync('./out/.env.email', 'utf-8')
    expect(compactString(content).replace(/\"/g,"")).toContain(compactString(generateSecrets(src, 'email-service')));
  });
  test('sms service secret', () => {
    expect(fs.existsSync('./out/.env.sms')).toBe(true);
    const src = fs.readFileSync('./src/secrets.yml', 'utf-8')
    const content = fs.readFileSync('./out/.env.sms', 'utf-8')
    expect(compactString(content).replace(/\"/g,"")).toContain(compactString(generateSecrets(src, 'sms-service')));
  });
  test('report service secret', () => {
    expect(fs.existsSync('./out/.env.report')).toBe(true);
    const src = fs.readFileSync('./src/secrets.yml', 'utf-8')
    const content = fs.readFileSync('./out/.env.report', 'utf-8')
    expect(compactString(content).replace(/\"/g,"")).toContain(compactString(generateSecrets(src, 'report-service')));
  });
});
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test("bashaway package functions should not be called", () => {
    expect(script).not.toContain("@sliit-foss");
    expect(script).not.toContain("bashaway");
  });
  test("no additional npm dependencies should be installed", async () => {
    await expect(dependencyCount()).resolves.toStrictEqual(3)
    expect(script).not.toMatch(prohibitedCommands);
  });
});
test('should check if there are hardcoded values', () => {
  const script = fs.readFileSync('./execute.sh', 'utf-8')
  expect(script).not.toContain("TWILIO");
  expect(script).not.toContain("MAIL_HOST");
  expect(script).not.toContain(".sms");
  expect(script).not.toContain(".env.email");
  expect(script).not.toContain(".report");
});
test('should not download external scripts', () => {
  const script = fs.readFileSync('./execute.sh', 'utf-8')
  expect(script).not.toContain("curl");
  expect(script).not.toContain("wget");
});
```

**Key idea:** Turn Kubernetes Secret YAML (multi-document) into `.env.<prefix>` files. With `yq` it's one loop. In pure bash it's a line-by-line state machine.

**Solution: team *TrackTech*** (pure-bash state machine; `yq` would be much shorter)
```bash
#!/bin/bash
mkdir -p out
regex="^[[:space:]]*[A-Z_]+:[[:space:]]*\S+"
kind=0
name=0
d=0
current_name=""
while IFS= read -r line; do
    if [[ $line == "#"* ]]; then
        continue
    fi
    if [[ $line == "" ]]; then
        continue
    fi
    if [[ $line == "kind: Secret"* ]]; then
        kind=1
        name=0
        d=0
        continue
    fi
    if [[ $line == *"name:"* ]]; then
        name_part="${line%%:*}"  # Extract "name"
        current_name=$(echo "$line" | cut -d':' -f2 | cut -d'-' -f1)
        current_name=$(echo $current_name | tr -d '[:space:]')
        touch "out/.env.$current_name"
        name=1
        continue
    fi
    if [[ $line == "stringData:"* ]]; then
        d=1
        continue
    fi
    if [[ $name == 1 && $d == 1 ]]; then
        line=$(echo "$line" | tr -s '[:space:]')
        if [[ "$line" =~ $regex ]]; then
            line=$(echo "$line" | cut -d'/' -f1)
            line=$(echo "$line" | sed 's/:/=/')
            echo "$line" >> "out/.env.$current_name"
        fi
    fi
done < src/secrets.yml
```

---

### exclusion (2023 · easy)

**Task**

Within a directory filled with mysterious text files bearing diverse names, your quest is to seek and **separate those untouched by the ungodly "534" pattern**. With your bash script as the wand, conjure a new realm called **out**, where only the unsullied files shall reside. Will you rise to this challenge?

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output**

- All of the text files which do not match the given pattern with the same content as the original in a directory named **out**

**Starter code**

Starter `execute.sh`: (empty)

*Project tree (27 files, 0 KB), shown 2 levels deep:*
```
src/0-99849234234.txt
src/0o90rt9rewt89.txt
src/12312342342365432.txt
src/123424453455.txt
src/23423454t4564677546345.txt
src/23442m3c0u584325.txt
src/32423443524545656.txt
src/32432r4323432 copy 22.txt
src/32432r4323432 copy.txt
src/32432r4323432.txt
src/342543r34rf3r232d234234.txt
src/35424fdsgcdfgdfg.txt
src/4343535345.txt
src/456456y6y3453g.txt
src/53445453442.txt
src/9883iiiiopsf+534dsf.txt
src/bfgdgh345.txt
src/c^fdsgtg345.txt
src/dfg3450353454534.txt
src/dfgdfsg2345345543.txt
src/e234234trt54354.txt
src/hbfghdhg8t34.txt
src/hrtt87.txt
src/ldfgsik9d0sfg.txt
src/mfafw234234msgsdf5341232sdfsdff.txt
src/ssdf-vcdsfgdf.txt
src/y435345234.txt
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; no additional npm dependencies should be installed; should check for javascript usage; should check for python usage; should validate if only bash files are present
test('should check if the output files have been filtered correctly', () => {
 expect(scanPure('out/**').length).toStrictEqual(scanPure('src/**').filter((file) => !file.includes('534')).length);
});
test('should check if the output files match the input files', () => {
 scanPure('out/**').forEach((file) => {
  expect(fs.readFileSync(file, 'utf-8')).toBe(fs.readFileSync(file.replace('out', 'src'), 'utf-8'));
 });
});
```

**Key idea:** `mkdir -p out; find src -type f ! -name '*534*' -exec cp {} out/ \;`. The test filters on the **file name/path** containing `534`, not on the file contents. (The repo's code/execute.sh uses `*5334*`, which is wrong.)

---

### fate-of-atlantis (2023 · easy)

**Fate of Atlantis**

In the midst of an ancient underwater war between the cities of Atlantis and Lemuria, there was a single, treasured log file within a chest named src. This log contained the secrets of numerous battles, with each entry marked by a unique correlation id, symbolizing the enemy's incursions.

In this epic struggle, the fate of Atlantis depended on uncovering a specific set of log entries that matched a given correlation code. The question was simple: Could they locate these vital entries within the log and decipher the enemy's plans, thereby tipping the scales in their favor in this ancient and ongoing battle for dominance?

Example invokation:-

```bash
bash execute.sh 243d8ebfd86602a2873cc671cc783540
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- The set of log entries that match the given correlation id should be printed to the console in the order they appear in the log file.

- The script should not crash if no correlation id is provided.

**Starter code**

Starter `execute.sh`: (empty)

*Project tree (1 files, 48 KB), shown 2 levels deep:*
```
src/logs.txt
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
describe('should search for logs', () => {
  let logs
  beforeAll(() => {
    logs = fs.readFileSync('./src/logs.txt', 'utf-8')
  });
  test('search for correlation id - 86e7f2d8f9ceb323e7386a4c33c7553e', async () => {
   const output = await exec(`bash execute.sh 86e7f2d8f9ceb323e7386a4c33c7553e`)
   expect(compactString(output?.trim())).toMatch(compactString(logs?.trim()?.split('\n').filter(log => log.includes('86e7f2d8f9ceb323e7386a4c33c7553e'))?.join('\n')))
  });
  test('search for correlation id - e943152799966996542a17fd674a4c1f', async () => {
    const output = await exec(`bash execute.sh e943152799966996542a17fd674a4c1f`)
    expect(compactString(output?.trim())).toMatch(compactString(logs?.trim()?.split('\n').filter(log => log.includes('e943152799966996542a17fd674a4c1f'))?.join('\n')))
  });
  test('search for correlation id - 6d13bb3452f06140772e6a69a167527f', async () => {
    const output = await exec(`bash execute.sh 6d13bb3452f06140772e6a69a167527f`)
    expect(compactString(output?.trim())).toMatch(compactString(logs?.trim()?.split('\n').filter(log => log.includes('6d13bb3452f06140772e6a69a167527f'))?.join('\n')))
  });
  test('search for correlation id - 86e7f2d8f9ceb323e7386a4c33c7553e', async () => {
    const output = await exec(`bash execute.sh 86e7f2d8f9ceb323e7386a4c33c7553e`)
    expect(compactString(output?.trim())).toMatch(compactString(logs?.trim()?.split('\n').filter(log => log.includes('86e7f2d8f9ceb323e7386a4c33c7553e'))?.join('\n')))
  });
});
```

**Key idea:** `[ -z "$1" ] && exit 0; grep -- "$1" src/logs.txt`. grep keeps the original line order.

---

### lockdown (2023 · easy)

**Lockdown**

In a distant land where enchanted barriers kept knowledge imprisoned, Alistair, a rogue wizard, found himself locked in a tower during the Great War. Desperate to preserve his forbidden knowledge, he cast a powerful spell on a sacred text, transforming it into an unalterable, read-only relic. Hidden within the **src** directory, the file became a tantalizing secret, waiting for a worthy seeker to unlock its secrets.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- The text file within the **src** directory must be restricted to read-only access.

**Starter code**

Starter `execute.sh`: (empty)

`src/relic.txt`:
```
Ingrdients for a magical elixir:

 - 1/2 cup of raw honey
 - 1/4 cup of dragon blood
 - a few drops of unicorn tears
 - a feather from a phoenix
 - a pinch of fairy dust
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const fs = require('fs/promises');
test('should check if the relic.txt file is read only', async () => {
  expect(fs.appendFile('./src/relic.txt', "abc")).rejects.toThrow();
  fs.appendFile('./src/relic.txt', "abc").catch((err) => {
    expect(['EPERM: operation not permitted, open', 'EACCES: permission denied, open'].find(m=> err.message.includes(m))).toBeTruthy();
  });
});
```

**Key idea:** `chmod 444 src/*` (read-only for everyone).

---

### log-cleaner (2023 · easy)

**Log cleaner**

Craft a script that, based on the file's language, purges all unwanted echoes from the codebase. Silence the chatter of **console.log in JavaScript**, hush the **print incantations in Python and Dart**, banish the **System.out.println cries in Java** and extinguish the **Console.WriteLine whispers in C#** restoring harmony to the code. The tarnished files can be found in the **src** directory.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- The cleaned files should be inside a directory called **out**

**Starter code**

Starter `execute.sh`: (empty)

`src/Math.cs`:
```csharp
using System;

public class Math
{
    private int _result;

    public int result {
       get {
           return _result;
       }
    }

    public Math(int startNum) {
        _result = startNum;
    }

    public void add(int num) {
        Console.WriteLine("Adding {0} to {1}", num, _result);
        _result += num;
    }

    public void subtract(int num) {
        Console.WriteLine("Subtracting {0} from {1}", num, _result);
        _result -= num;
    }

    public void multiply(int num) {
        Console.WriteLine("Multiplying {0} with {1}", num, _result);
        _result *= num;
    }

    public void divide(int num) {
        Console.WriteLine("Dividing {0} from {1}", num, _result);
        _result /= num;
    }
}
```

`src/Seeder.java`:
```java
package app.database.seeders;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class Seeder implements CommandLineRunner {

    @Autowired
    private CitySeeder citySeeder;

    @Autowired
    private RoleSeeder roleSeeder;

    @Autowired
    private UserSeeder userSeeder;

    @Override
    public void run(String... args) throws Exception {
        System.out.println("Database seeding started...");

        citySeeder.seed();
        System.out.println("Cities seeded.");

        roleSeeder.seed();
        System.out.println("Roles seeded.");

        userSeeder.seed();
        System.out.println("Users seeded.");

        System.out.println("Database seeding completed.");
    }
}
```

`src/cache.dart` (2074 bytes, 60 lines, first 8 lines shown):
```dart
import 'dart:convert';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class Cache {
  static const storage = FlutterSecureStorage();
  static const aOptions = AndroidOptions();
  static const iOptions = IOSOptions(accessibility: KeychainAccessibility.first_unlock);

...
```

`src/index.js` (2034 bytes, 67 lines, first 8 lines shown):
```js
const _fnName = (fn) => {
    let name = fn.name;
    while (1) {
        const replaced = name?.replace("bound", ""); 
        if (name.startsWith("bound") && replaced?.startsWith(" "))(name = replaced?.trim());
        else break;
    }
    if (!name) return "Unnamed function";
...
```

`src/mystery.py`:
```python
import random

def mysterious_function(value):
    """
    Perform mysterious operations on the input value.

    Parameters:
        value: Input value for mysterious operations.

    Returns:
        str: A mysterious string generated based on the input value.
    """
    print("Performing initial verification...")

    if not isinstance(value, int):
        print("Invalid input. Please provide an integer.")
        return None

    print("Performing advanced calculations...")

    mysterious_result = value * random.randint(1, 10)

    print("Generating mystical string...")

    mystic_string = ""
    for i in range(value):
        mystic_string += chr(random.randint(65, 90))

    print("Finalizing the mystery...")

    result = f"{mystic_string} - {mysterious_result}"

    print("Mystery complete!")
    return result

mysterious_function(5)
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; no additional npm dependencies should be installed; should validate if only bash files are present
test('should check if there are 5 output files', () => {
  expect(scanPure('./out/**').length).toBe(5);
});
describe('should check if the logs are removed', () => {
  test('dart', () => {
    expect(fs.existsSync('./out/cache.dart')).toBe(true);
    const src = fs.readFileSync('./src/cache.dart', 'utf-8')
    const content = fs.readFileSync('./out/cache.dart', 'utf-8')
    expect(compactString(content)).toStrictEqual(cleanLogs(src))
  });
  test('java', () => {
    expect(fs.existsSync('./out/Seeder.java')).toBe(true);
    const src = fs.readFileSync('./src/Seeder.java', 'utf-8')
    const content = fs.readFileSync('./out/Seeder.java', 'utf-8')
    expect(compactString(content)).toStrictEqual(cleanLogs(src))
  });
  test('csharp', () => {
    expect(fs.existsSync('./out/Math.cs')).toBe(true);
    const src = fs.readFileSync('./src/Math.cs', 'utf-8')
    const content = fs.readFileSync('./out/Math.cs', 'utf-8')
    expect(compactString(content)).toStrictEqual(cleanLogs(src))
  });
  test('python', () => {
    expect(fs.existsSync('./out/mystery.py')).toBe(true);
    const src = fs.readFileSync('./src/mystery.py', 'utf-8')
    const content = fs.readFileSync('./out/mystery.py', 'utf-8')
    expect(compactString(content)).toStrictEqual(cleanLogs(src))
  });
  describe('javascript', () => {
    test('console.logs removed', () => {
      expect(fs.existsSync('./out/index.js')).toBe(true);
      const src = fs.readFileSync('./src/index.js', 'utf-8')
      const content = fs.readFileSync('./out/index.js', 'utf-8')
      expect(compactString(content)).toStrictEqual(cleanLogs(src))
    });
    test('script should execute', async () => {
      await expect(exec("node ./out/index.js")).resolves.toContain("Hello World");
    });
  })
});
test('should check if there are hardcoded values', () => {
  const script = fs.readFileSync('./execute.sh', 'utf-8')
  expect(script).not.toContain("package:flutter_secure_storage/flutter_secure_storage.dart");
  expect(script).not.toContain("process.env.DISABLE_FUNCTION_TRACING");
  expect(script).not.toContain("using System;");
  expect(script).not.toContain("import random");
  expect(script).not.toContain("org.springframework.boot.CommandLineRunner");
});
test('should not download external scripts', () => {
  const script = fs.readFileSync('./execute.sh', 'utf-8')
  expect(script).not.toContain("curl");
  expect(script).not.toContain("wget");
});
```

**Key idea:** Delete lines per language with `sed '/pattern/d'`. For multi-line JS `console.log(...)` calls, slurp the whole file with `sed ':a;N;$!ba;...'` and then substitute.

**Solution: team *SLIIT FOSS***
```bash
mkdir -p out
sed '/print/d' src/cache.dart > out/cache.dart
sed '/Console.WriteLine/d' src/Math.cs > out/Math.cs
sed '/print/d' src/mystery.py > out/mystery.py
sed '/System.out.println/d' src/Seeder.java > out/Seeder.java
sed ':a;N;$!ba;s/console\.log([^)]*)[^;]*;//g' src/index.js > out/index.js
```

---

### martian-pdf (2023 · easy)

**Martian PDF**

On the enigmatic red planet of Mars, where ancient scrolls of knowledge are inscribed in strange hieroglyphs, imagine stumbling upon a peculiar Martian PDF. Its pages are like fragments of Martian history, filled with alien symbols and cryptic runes. Your mission, as a galactic explorer, is to decode the secrets of a specific Martian page using the mystical language of Bash.

As you venture through the Martian PDF, your Bash skills act as an interplanetary Rosetta Stone, deciphering the word count of the chosen page. Just as Martians might decode the mysteries of Earth's languages, you'll unveil the Martian page's word count, bridging the gap between worlds with your digital prowess.

**Example Invocation**

```bash
bash execute.sh 1 # 1st page
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A **single integer to the console** representing the **word count of a specific page of the given PDF file named mystery.pdf** which can be found within the **src** directory.

- If the page does not exist, the script should output **0**.

- The script should not crash if the src directory is empty or does not exist or if a page number is not provided.

**Starter code**

Starter `execute.sh`: 
```bash
# A pdf file will be generated within a src directory once the tests are run.
# Write your code here
```

Extra npm deps used by the tests: `pdfkit@0.13.0`

*Project tree (1 files, 38 KB), shown 2 levels deep:*
```
src/mystery.pdf
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const PDFDocument = require('pdfkit');
test('should check if the word count is obtained properly', async () => {
  if (!fs.existsSync('./src')) fs.mkdirSync('./src');
  const doc = new PDFDocument();
  doc.pipe(fs.createWriteStream('./src/mystery.pdf'));
  const pageContent = {
    1: faker.lorem.paragraph({ min: 3, max: 20 })
  }
  let page = 1;
  doc.text(pageContent[1]);
  for (let i = 0; i < faker.number.int({ min: 35, max: 60 }); i++) {
    const content = faker.lorem.paragraph({ min: 3, max: 20 });
    pageContent[++page] = content;
    doc.addPage().text(content);
  }
  doc.end();
  const testPage = faker.number.int({ min: 1, max: 40 });
  const result = await exec(`bash execute.sh ${testPage}`);
  console.log(Number(result?.trim()), pageContent[testPage]?.split(' ')?.length)
  expect(Number(result?.trim())).toStrictEqual(pageContent[testPage]?.split(' ')?.length ?? 0);
});
```

**Key idea:** `pdftotext -f N -l N file.pdf - | wc -w` (from poppler-utils). Guard against a missing dir, file, or argument and print 0.

**Solution: team *' OR 1=1 --***
```bash
if [ -d "src" ] && [ -f "src/mystery.pdf" ]; then
    if [ -z "$1" ]; then
        echo 0
    else
        pdftotext -f $1 -l $1 src/mystery.pdf - | wc -w
    fi
else
    echo 0
fi
```

---

### mutate-x (2023 · easy)

**Task**

Bestow the sacred **X** upon the file known as **mutate.txt** dwelling within the **src** directory. After this transformation, usher a copy of this file into the mystical domain of **out**. All shall be accomplished through the arcane arts of bash, as the only true path.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output**

- A single text file named **mutateX** with the same content as the original in a directory named **out**

**Starter code**

Starter `execute.sh`: (empty)

`src/mutate.txt`:
```
Just a sample text file
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; no additional npm dependencies should be installed; should validate if only bash files are present
test('should check if there is only one output file', () => {
 expect(scanPure('./out/**').length).toBe(1);
});
test('should check if the output file name is mutateX.txt', () => {
 expect(scanPure('./out/**')[0].endsWith('mutateX.txt')).toBe(true);
});
```

**Key idea:** The test only checks that `out/mutateX.txt` exists with the same content: `mkdir -p out; cp src/mutate.txt out/mutateX.txt`.

---

### mystic-force (2023 · easy)

**Mystic Force**

In the ancient days of stone tools and cave dwellers, imagine a world where knowledge was etched onto primitive slabs of rock. Now, picture a mystical stone tablet named **a.txt** that held the secrets of the tribe. To ensure the tribe's wisdom was preserved, a shaman duplicated this tablet into **b.txt.** Here's the catch: **whenever a tribal elder inscribed new knowledge onto "a.txt" using their crude markings, the same mystic force engraved those changes onto "b.txt" as well**, as if guided by the spirits of the ancients. Now, in the digital age, we seek to replicate this ancient magic with modern files - creating a.txt and b.txt, linked in such a way that any change to a.txt is mirrored in b.txt, just like the mystical connection of those ancient stone tablets.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

- The script must be **less than 30 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- A copy of a.txt named b.txt created in the same directory as a.txt (src) and linked to it in such a way that any change to a.txt is mirrored in b.txt.

**Starter code**

Starter `execute.sh`: (empty)

`src/a.txt`:
```
Haylow
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
jest.setTimeout(180000);
afterAll(() => {
  fs.writeFileSync('./src/a.txt', 'Haylow');
});
test('should check if a.txt is duplicated and linked properly', async () => {
  expect(fs.readFileSync('./src/b.txt', 'utf-8')).toBe('Haylow');
  fs.appendFileSync('./src/a.txt', ' World!');
  expect(fs.readFileSync('./src/b.txt', 'utf-8')).toBe('Haylow World!');
});
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 30 characters in length', () => {
    expect(script.length).toBeLessThan(30);
  });
});
```

**Key idea:** Make a hard link or a symlink: `ln src/a.txt src/b.txt` or `ln -s a.txt src/b.txt`. A relative symlink target is resolved from the link's own directory. The limit is under 30 characters.

---

### numerological-incantation (2023 · easy)

**Numerological Incantation**

In the mystical land of Numeria, Master Emeric, a humble scribe and mathematician, discovered the ancient Scroll of Prime Mysteries. Legend has it that this scroll contains the key to unlocking the secrets of numbers in the realm. Tasked with a noble quest by the kingdom's wise wizard, Emeric must create a bash script, akin to a magical incantation, to **represent any number's prime factors in the sacred format of f1 x f2 x f3 x fn** and unlock the ancient numerological secrets of Numeria.

**Example Invokation**

```bash
bash execute.sh 12
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A single line to the console that represents the prime factors of the number passed in as the first argument to the script. The output should be in the format of f1 x f2 x f3 x fn, where each f is a prime factor of the number passed in. If the number is prime, the output should be the number itself.

- The script should not crash if no number is passed in as an argument. Instead, it should print a message to the console that says "Please pass in a number as an argument to the script."

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const fs = require('fs/promises');
jest.setTimeout(45000)
test('should check if the prime factors are obtained properly', async () => {
  for (let i = 0; i < 10; i++) {
    const number = faker.number.int({ min: 150, max: 500000 });
    const result = await exec(`bash execute.sh ${number}`);
    expect(result?.trim()?.split('x').reduce((a, b) => +a * +b, 1)).toBe(number);
  }
});
test('should check invokation with no arguments', async () => {
  const result = await exec('bash execute.sh');
  expect(result?.trim()).toStrictEqual('Please pass in a number as an argument to the script.');
});
```

**Key idea:** Coreutils `factor` prints `12: 2 2 3`. Re-join the factors with ` x ` using awk.

**Solution: team *SLIIT FOSS***
```bash
if [ -z $1 ]; then
    echo "Please pass in a number as an argument to the script."
    exit 0
fi
factor $1 | awk '{for(i=2; i<=NF; i++) {printf("%s%s", sep, $i); sep=" x "}; printf("\n")}'
```

---

### servant-of-darkness (2023 · easy)

**Servant of Darkness**

Far far away within the caverns of the Soulforge, a demon has been summoned. This demon is quite powerful as it is hungry. You as its herald is tasked with feeding it with the souls of the innocent. The demon is quite picky and will only accept souls that are stored within the Azure Realm which unfortunately is millions of light years away. You must find a way to feed the demon with the souls of the innocent by emulating the existence of Azure within the Soulforge.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- Run a local instance of Azure blob storage on port **12000** on your machine so that the tests can be run against it.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `@azure/storage-blob@12.16.0`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
const { BlobServiceClient, StorageSharedKeyCredential } = require("@azure/storage-blob");
jest.setTimeout(15000);
describe('should create and download a blob successfully', () => {
  const containerName = 'bashaway';
  const blobServiceClient = new BlobServiceClient(
    `http://devstoreaccount1.blob.core.windows.net:12000`,
    new StorageSharedKeyCredential("devstoreaccount1", "Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==")
  );
  let containerClient;
  beforeAll(async () => {
    containerClient = blobServiceClient.getContainerClient(containerName);
    if (!await containerClient.exists()) await containerClient.create().catch(console.log);
  });
  afterAll((done) => done());
  const streamToString = async (stream) => {
    const chunks = [];
    for await (const chunk of stream) {
      chunks.push(Buffer.from(chunk));
    }
    return Buffer.concat(chunks).toString("utf-8");
  }
  const fileName = faker.system.fileName();
  const fileContent = faker.string.alphanumeric(500)
  test('should create a blob successfully', async () => {
    const blockBlobClient = containerClient.getBlockBlobClient(fileName);
    const uploadBlobResponse = await blockBlobClient.upload(fileContent, fileContent.length);
    expect(uploadBlobResponse._response.status).toBe(201);
  });
  test('should download a blob successfully', async () => {
    const blockBlobClient = containerClient.getBlockBlobClient(fileName);
    const downloadBlockBlobResponse = await blockBlobClient.download(0);
    const downloadedContent = await streamToString(downloadBlockBlobResponse.readableStreamBody);
    expect(downloadedContent).toBe(fileContent);
  });
  test('should delete a blob successfully', async () => {
    const blockBlobClient = containerClient.getBlockBlobClient(fileName);
    const deleteBlobResponse = await blockBlobClient.delete();
    expect(deleteBlobResponse._response.status).toBe(202);
  });
});
```

**Key idea:** Use Azurite (the Azure Storage emulator) in Docker and map host port 12000 to 10000. Point the account hostname at 127.0.0.1 in `/etc/hosts`.

**Solution: team *HereForTheBytes***
```bash
echo "127.0.0.1 	devstoreaccount1.blob.core.windows.net" | sudo tee -a /etc/hosts
docker run -d -p 12000:10000 mcr.microsoft.com/azure-storage/azurite
```

---

### spellbound (2023 · easy)

**Spellbound**

In a realm where code and sorcery intertwined, an enigmatic coder faced a task reminiscent of a witcher's quest. **Within the src directory lay a cryptic JSON grimoire, filled with objects bearing unpredictable keys and values**. His mission was to **reveal the object where the key "name" held the elusive word "bashaway."** With a spellbound script, he ventured through the arcane data, and like a seasoned witcher, he tracked down the object and its position.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- The array index of the object where the key "name" holds the value "bashaway" must be printed to the console.

- The script should not crash if the src directory is empty or does not exist.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javascript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
let insertedIndex = -1;
beforeAll(async () => {
  if (fs.existsSync('./src')) fs.rmSync('./src', { recursive: true });
  fs.mkdirSync('./src');
  const arr = Array.from({ length: faker.number.int({ min: 500, max: 1000 }) },
  () => Array(faker.number.int({ min: 5, max: 10 })).fill(0).reduce((acc) => {
    acc[faker.word.noun()] = faker.word.adjective();
    return acc;
  }, {}));
  insertedIndex = faker.number.int({ min: 0, max: arr.length })
  arr.splice(insertedIndex, 0, { name: 'bashaway' });
  fs.writeFileSync('./src/grimoire.json', JSON.stringify(arr));
});
test('should check if the index is obtained properly', async () => {
  const result = await exec('bash execute.sh');
  expect(Number(result)).toStrictEqual(insertedIndex);
});
```

**Key idea:** Use jq to find the array index: `to_entries[] | select(.value.name=="bashaway") | .key`.

**Solution: team *SLIIT FOSS***
```bash
sudo apt-get install jq
if [ ! -f src/grimoire.json ]; then
    echo "No data to parse. Exiting..."
    exit 0
fi
echo $(cat src/grimoire.json | jq '[.[].name] | to_entries | .[] | select(.value=="bashaway") | .key')
```

---

### stone-age (2023 · easy)

**Stone age**

In a time travel mishap, a modern programmer named Alex found himself back in the Stone Age. He discovered cave markings resembling a simple programming language, much like Pascal. Using primitive tools, Alex configured an environment, executed the markings, and witnessed patterns of smoke rise from a makeshift console possibly conveying important survival instructions to ancient inhabitants. Alex's unusual encounter showcased the power of ancient knowledge, leaving a lasting impact on the Stone Age society.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

- You cannot read from the input file and echo anything to the console. You must execute the pascal script for it.

**Output / Evaluation Criteria**

- Execute and get the output of the pascal script given within the src directory on the console

- Make sure that you clean up the directory after any necessary installations or downloads

**Starter code**

Starter `execute.sh`: (empty)

`src/markings.pas`:
```pascal
program markings; 
begin
   writeln('Delectus voluptas autem. Dolores provident dolor esse a vel quisquam. A facilis qui voluptatum fugit libero similique.');
   writeln('253');
end.
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used
jest.setTimeout(150000)
test('should check if the output is obtained from the pascal script', async () => {
  const text = faker.lorem.paragraph();
  const hex = faker.number.hex({ min: 1, max: 1000 })
  const int = faker.number.int({ min: 1, max: 1000 })
  const binary = faker.number.binary({min: 400, max: 4000})
  fs.writeFileSync('./src/markings.pas', `program markings;    \n\nbegin\n\n\n      writeln('${text}');\n   writeln('${hex}'); // this is a hex \n
(* This is a binary file *)
  writeln(${binary});\n
  {  This is a int }
  writeln('${int}'); // this is an int \n
  end.`);
  const result = await exec('bash execute.sh');
  expect(result?.trim()).toContain(text);
  expect(result?.trim()).toContain(hex.toString());
  expect(result?.trim()).toContain(int.toString());
  expect(result?.trim()).toContain(binary.toString());
  expect(result?.trim()).not.toContain('writeln');
  expect(result?.trim()).not.toContain('program markings;');
});
describe('should check constraints', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test("nothing should be echoed or printed to the console", () => {
    expect(script).not.toContain("echo");
    expect(script).not.toContain("print");
  });
  test("cannot read or modify file the input file", () => {
    expect(script).not.toContain("cat");
    expect(script).not.toContain("awk");
    expect(script).not.toContain("sed");
  });
});
```

**Key idea:** Install a compiler or interpreter (`apt-get install -y fpc`), compile, run, and **clean up the binary and object files** afterwards.

**Solution: team *SLIIT FOSS*** (doesn't clean up the `.o` file and binary. Add `rm` at the end.)
```bash
sudo apt-get update
sudo apt-get install fpc
fpc src/markings.pas
./src/markings
```

---

### system-insight (2023 · easy)

**System Insight**

Once upon a time, a brave coder embarked on a quest to uncover the secrets of a mysterious machine. Armed with their trusty terminal, they delved into the heart of the system, determined to reveal its hidden treasures.

With the incantation of commands and the wave of their keyboard, the coder summoned the CPU's true name, whispered by the spirits of technology. They counted the cores, like counting stars in the night sky, and measured the memory, capturing its essence in bytes.

As the final piece of the puzzle, they unearthed the machine's true identity, its hostname, a secret name known only to the machine itself.

In the end, the brave coder triumphed, presenting their findings in a magical JSON scroll that revealed the machine's secrets to the world:

```json
{
  "cpu": "Intel(R) Core(TM) i7-7700HQ CPU @ 2.80GHz",
  "cores": 12,
  "memory": "47.9 GB",
  "hostname": "DESKTOP-FMO5HV6"
}
```

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

- The script must be **less than 300 characters** in length. (Including spaces, newlines, and the shebang line)

**Output / Evaluation Criteria**

- The above JSON must be printed to the console with accurate values.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const os = require('os');
test('should check if the system info is obtained and printed to the console', async () => {
  let output = await exec(`bash execute.sh`);
  expect(JSON.parse(output)).toEqual({
    cpu: os.cpus()[0].model,
    cores: os.cpus().length,
    memory: `${(os.totalmem() / 1024 / 1024 / 1024).toFixed(1)} GB`,
    hostname: os.hostname(),
  })
});
describe('should check installed dependencies', () => {
  let script
  beforeAll(() => {
    script = fs.readFileSync('./execute.sh', 'utf-8')
  });
  test('the script should be less than 300 characters in length', () => {
    expect(script.length).toBeLessThan(300);
  });
});
```

**Key idea:** `lscpu` or `/proc/cpuinfo` for the CPU model, `nproc` for cores, `/proc/meminfo` MemTotal (kB → GB) for memory, `hostname`. Print it as JSON with `printf`. The limit is under 300 characters.

**Solution: team *iCoders***
```bash
#!/bin/bash
echo "{\"cpu\":\"$(lscpu | grep 'Model name' | cut -d ':' -f2 | xargs)\",\"cores\":$(nproc),\"memory\":\"$(awk '/MemTotal/ {printf "%.1f GB", $2/1024/1024}' /proc/meminfo)\",\"hostname\":\"$(hostname)\"}"
```

---

### wartime-secrets (2023 · easy)

**Wartime Secrets**

Amidst the backdrop of colonial India, a brilliant mathematician was on a secret mission to decode enemy messages during World War II. He had to design a password system to secure critical information. Assume you are that mathematician and your challenge here is to create a Bash script that can determine if a password is as strong as the cryptographic defenses of that time. Can your code match the strength of those wartime secrets?

**Constraints**

- The script must be **purely written in bash within the execute.sh file**

**Output / Evaluation Criteria**

- A single **true** or **false** to the console if the password is strong enough or not. A password is considered strong enough if it meets all of the following conditions:
  - It contains at least 1 lowercase alphabetical character
  - It contains at least 1 uppercase alphabetical character
  - It contains at least 1 numeric character
  - It is at least 8 characters long

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
jest.setTimeout(10000)
test('should check if the password is validated properly', async () => {
  for (const _ of Array(25).keys()) {
    const randomPassword = faker.internet.password({ length: faker.number.int({ min: 1, max: 15 }), pattern: /[\dA-Za-z]/ });
    const valid = await exec(`bash execute.sh \'${randomPassword}\'`);
    expect(valid?.trim()).toBe((!!isStrongPassword(randomPassword)).toString());
  }
});
```

**Key idea:** Several `[[ $p =~ [a-z] ]]` regex tests plus `${#p} -ge 8`.

**Solution: team *5H4D0W G4RD3N***
```bash
#!/bin/bash
password="$1"
if [[ "$password" =~ [a-z] && "$password" =~ [A-Z] && "$password" =~ [0-9] && ${#password} -ge 8 ]]; then
    echo "true"
else
    echo "false"
fi
```

---

### webcraft (2023 · easy)

**WebCraft**

Prepare for a mesmerizing adventure in the realm of the undying web servers. Your mission: to **host a magical HTML page grounded within src** on your local machine, beckoning it to life through the sacred portal of **port 8085**.

**Constraints**

- The script must be **purely written within the execute.sh file**.
- You can install any tool you need to complete the task

**Output / Evaluation Criteria**

- http://localhost:8085 must serve the given HTML page

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `axios@1.5.0`

`src/page.html` (2292 bytes, 106 lines, first 8 lines shown):
```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <link rel="icon" type="image/svg+xml" href="https://bashaway.sliitfoss.org/assets/icons/favicon.svg" />
  <title>Webcraft | Bashaway 2023</title>
...
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
const axios = require('axios').default;
test("should check if the webpage is being served properly", async () => {
  const content = fs.readFileSync('src/page.html', 'utf8');
  const response = await axios.get('http://localhost:8085');
  expect(response.data).toBe(content);
});
```

**Key idea:** Any static server works, as long as it runs in the background with `&`: `php -S`, `python3 -m http.server 8085 -d src`, `npx serve`, or nginx in Docker. If `/` must return the page, name it `index.html` or pass it as the router script.

**Solution: team *REISUB***
```bash
cd src
php -S localhost:8085 page.html &
```

---

---

## 2023 · MEDIUM

### arcane-alchemy (2023 · medium)

**Arcane Alchemy**

In a world brimming with possibilities, where data takes on a myriad forms, undertake an a quest to **metamorphose an unpredictable XML parchment, concealed within the src directory, into a JSON artifact** of formidable power. Harness your coding prowess to perform this arcane alchemy, and then set forth on an epic odyssey to **safeguard this newly wrought gem within the out directory**, where its transformative abilities shall fuel future adventures beyond imagination!

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A single JSON file, with the same name as the XML file, should be created within the out directory.

- The script should not crash if the src directory is empty or does not exist.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `xml-js@1.6.11`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javascript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const convert = require('xml-js');
let filename = null, data = null;
beforeAll(async () => {
  if (fs.existsSync('./src')) fs.rmSync('./src', { recursive: true });
  fs.mkdirSync('./src');
  filename = faker.word.noun();
  data = {
    [faker.word.noun()]: Array(faker.number.int({ min: 25, max: 50 })).fill(0).reduce((acc) => {
      acc[faker.word.noun()] = faker.word.adjective();
      return acc;
    }, {})
  }
  fs.writeFileSync(`./src/${filename}.xml`, convert.json2xml(data, { compact: true, ignoreComment: true, spaces: 4 }));
});
test('should check if the xml file is converted properly to JSON', async () => {
  await exec('bash execute.sh');
  expect(JSON.stringify(JSON.parse(fs.readFileSync(`./out/${filename}.json`, 'utf-8')))).toStrictEqual(JSON.stringify(data));
});
```

**Key idea:** Convert XML to JSON with mikefarah `yq --input-format xml --output-format json` (or `-p xml -o json`). Install it as a single binary from GitHub releases.

**Solution: team *SLIIT FOSS***
```bash
if [ ! -d "./src" ]; then
  echo "No data to convert. Exiting..."
  exit 0
fi
sudo wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/bin/yq && sudo chmod +x /usr/bin/yq
file=$(basename -- $(find src -name "*.xml") .xml)
rm -rf out && mkdir -p out
yq --input-format xml --output-format json ./src/$file.xml >> out/$file.json
```

---

### cluttered (2023 · medium)

**Cluttered**

In the vast landscape of MongoDB's data kingdom named **bashaway-2k23-cluttered**, your noble quest beckons—a quest to wield the script of purification. As the guardian of order, your solemn charge is clear: to sweep away the clutter and restore serenity to this digital realm. Let all collections bow before your script, save for the revered "settings" collection, which shall remain untouched, a beacon of knowledge amidst the cleansing tide. Thus, you, the script-weaver, shall bring balance to MongoDB's sprawling dominion.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- No collections except **settings** should be present in the database after the script is executed.

**Starter code**

Starter `execute.sh`: 
```bash
docker pull mongo:latest
if [ ! "$(docker ps -a | grep bashaway-2k23-cluttered)" ]; then
    docker run -d --name bashaway-2k23-cluttered -p 27207:27017 mongo
fi
# Data will be populated into the above database instance once the tests are run.
# Write your code here
```

Extra npm deps used by the tests: `mongodb@6.0.0`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; no additional npm dependencies should be installed; should validate if only bash files are present
const { client, insert, databaseName } = require('./insert');
jest.setTimeout(20000);
describe('should check if database is being cleaned properly', () => {
  let data
  beforeAll(async () => {
    data = await insert();
  });
  afterAll((done) => {
    client.close().then(done);
  });
  test('should check if the database is cleaned properly', async () => {
    const db = client.db(databaseName);
    const collections = await db.listCollections().toArray();
    expect(collections.length).toBe(21);
    await exec('bash execute.sh');
    const collectionsAfter = await db.listCollections().toArray();
    expect(collectionsAfter.length).toBe(1);
    const settings = await db.collection('settings').find().toArray();
    expect(settings.map((setting) => ({ key: setting.key, value: setting.value }))).toStrictEqual(data[0].documents);
  });
});
```

**Test `test/insert.js`**
```js
const mongodb = require('mongodb');
const { faker } = require('@faker-js/faker');

const data = [
    {
        collection: 'settings',
        documents: Array.from({ length: 10 }, () => ({
            key: faker.lorem.word(),
            value: faker.lorem.word()
        }))
    },
    ...Array.from({ length: 20 }, (_, index) => ({
        collection: `${faker.lorem.word()}-${index}`,
        documents: Array.from({ length: faker.number.int({ min: 1, max: 100 }) }, () => ({
            key: faker.lorem.word(),
            value: faker.lorem.word()
        }))
    }))
];

exports.databaseName = 'bashaway-2k23-cluttered';

exports.client = new mongodb.MongoClient('mongodb://localhost:27207/?directConnection=true');

exports.insert = async () => {
    await this.client.connect()

    const db = this.client.db(this.databaseName);

    db.dropDatabase();

    await Promise.all(JSON.parse(JSON.stringify(data)).map(({ collection, documents }) => db.collection(collection).insertMany(documents)))

    return data;
};

const args = process.argv.slice(2);

if (args.includes('--run')) {
    this.insert().then(() => {
        console.log('Data inserted successfully');
        process.exit(0);
    });
}
```

**Key idea:** Run `mongosh --eval` inside the container: loop over `db.getCollectionNames()` and drop every collection except `settings`.

**Solution: team *' OR 1=1 --***
```bash
docker pull mongo:latest
docker run -d --name bashaway-2k23-cluttered -p 27207:27017 mongo
sleep 5
docker exec bashaway-2k23-cluttered mongosh --eval 'db.getCollectionNames().forEach(function(collection) { if (collection !== "settings") db[collection].drop() });' bashaway-2k23-cluttered
```

---

### ethereal (2023 · medium)

**Ethereal**

Embark on an epic odyssey in the mystical kingdom of Cybersylvan, where a fabled **command of labyrinthine complexity, known as the etherealbinarybeast** has confounded even the wisest of sages. This formidable incantation, a sentence of code that stretched as far as the horizon, was whispered only by the boldest of coders.

Your valorous quest, noble scriptsmith, is to traverse the digital realms and **craft a celestial shortcut**, a command to rule them all, one that shall beckon the Invocation with but a humble utterance. This enchanted shortcut shall henceforth be celebrated as **ethereal**

Prepare thyself to unlock the deepest arcane secrets of the command-line arts, for you are the chosen one. Your journey shall be nothing short of legendary, casting a shimmering light across the tapestry of Cybersylvan!

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- Invoking the command **ethereal** should invoke the command **etherealbinarybeast**.

- The command **etherealbinarybeast** will only be established once the tests are executed. You need to handle the case where the command does not exist and the script should not crash if executed without the command being established.

**Starter code**

Starter `execute.sh`: 
```bash
# etherealbinarybeast command will be available once the tests are run
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
jest.setTimeout(60000)
let output = '';
beforeAll(async () => {
  if (!fs.existsSync('./src')) fs.mkdirSync('./src');
  output = faker.lorem.sentence();
  fs.writeFileSync('src/script.sh', `#!/bin/bash \n \n echo "${output}"`);
  fs.writeFileSync('src/package.json',JSON.stringify( {
    "name": "etherealbinarybeast",
    "version": "0.0.0",
    "bin": "script.sh"
  }));
  const currentDir = process.cwd();
  process.chdir('src');
  await exec('npm pack');
  await exec('bash -c \"sudo npm i -g etherealbinarybeast-0.0.0.tgz\"')
  fs.rmSync('etherealbinarybeast-0.0.0.tgz')
  fs.rmSync('package.json')
  fs.rmSync('script.sh')
  process.chdir(currentDir);
});
test('should check if the shortcut is setup properly', async () => {
  await exec('bash execute.sh');
  const result = await exec('bash -c "ethereal"');
  expect(result?.trim()).toStrictEqual(output);
});
```

**Key idea:** Create a command alias that persists outside your shell with `ln -s $(command -v etherealbinarybeast) /usr/local/bin/ethereal`, or a wrapper script. Guard with `command -v` so it doesn't crash when the target is missing.

**Solution: team *REISUB***
```bash
#!/bin/bash
command="etherealbinarybeast"
shortcut="ethereal"
if command -v $command &> /dev/null
then
    sudo ln -s $(which $command) /usr/local/bin/$shortcut
else
    echo "The command '$command' does not exist."
fi
```

---

### new-home (2023 · medium)

**A New Home**

Get ready to orchestrate a seamless data migration spectacle! In this challenge, you'll be the maestro of MongoDB as you conduct a symphony of **data transfer from one instance**, filled with valuable information, **to its barren counterpart**. Can you compose the perfect score for this, ensuring all the data finds a new home in the second instance?

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- All databases, collections and documents except for the system defaults which exist in the first instance (`bashaway-2k23-sunset`) must exist in the second instance (`bashaway-2k23-sunrise`)

- No extra databases, collections or documents should exist in the second instance (`bashaway-2k23-sunrise`) which do not exist in the first instance (`bashaway-2k23-sunset`)

- Make sure you clean up the working directory after the script has been executed. The working directory should be in the same state as it was before the script was executed.

**Starter code**

Starter `execute.sh`: 
```bash
docker pull mongo:latest
if [ ! "$(docker ps -a | grep bashaway-2k23-sunset)" ]; then
    docker run -d --name bashaway-2k23-sunset -p 27207:27017 mongo # Data will be populated into this instance once the tests are run.
fi
if [ ! "$(docker ps -a | grep bashaway-2k23-sunrise)" ]; then
    docker run -d --name bashaway-2k23-sunrise -p 27208:27017 mongo # Target instance to migrate data to.
fi
# Write your code here
```

Extra npm deps used by the tests: `mongodb@6.0.0`

`jest.config.js`:
```js
module.exports = {
    maxWorkers: 1
};
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; no additional npm dependencies should be installed; should validate if only bash files are present
const mongodb = require('mongodb');
jest.setTimeout(60000);
describe('populate data and check if it is cloned properly', () => {
  let insertedDatabases = null;
  const originInstance = new mongodb.MongoClient('mongodb://localhost:27207/?directConnection=true');
  const targetInstance = new mongodb.MongoClient('mongodb://localhost:27208/?directConnection=true');
  beforeAll(async () => {
    await Promise.allSettled([
      exec('docker run -d --name bashaway-2k23-sunset -p 27207:27017 mongo'),
      exec('docker run -d --name bashaway-2k23-sunrise -p 27208:27017 mongo')
    ])
    insertedDatabases = Array.from({ length: 5 }, (_, index) => ({
      database: `${faker.lorem.word()}-${index}`,
      collections: Array.from({ length: 10 }, (_, index) => ({
        name: `${faker.lorem.word()}-${index}`,
        documents: Array.from({ length: faker.number.int({ min: 1, max: 25 }) }, () => ({
          key: faker.lorem.word(),
          value: faker.lorem.word()
        }))
      }))
    }))
    await originInstance.connect();
    const admin = targetInstance.db().admin();
    const existingDatabases = (await admin.listDatabases())?.databases || [];
    await Promise.all(existingDatabases.map(({ name }) => {
      if (!["admin", "config", "local"].includes(name)) originInstance.db(name).dropDatabase();
    }));
    await Promise.all(insertedDatabases.map(({ database, collections }) => {
      const db = originInstance.db(database);
      return Promise.all(collections.map(({ name, documents }) => db.collection(name).insertMany(documents)));
    }))
  });
  afterAll((done) => {
    Promise.all([originInstance.close(), targetInstance.close()]).then(() => done());
  });
  test('should check if all of the data from the origin instance exists in the target instance', async () => {
    await targetInstance.connect();
    const admin = targetInstance.db().admin();
    const existingDatabases = (await admin.listDatabases())?.databases || [];
    await Promise.all(existingDatabases.map(({ name }) => {
      if (!["admin", "config", "local"].includes(name)) targetInstance.db(name).dropDatabase();
    }));
    await exec('bash execute.sh');
    const databases = (await admin.listDatabases()).databases.filter(({ name }) => !["admin", "config", "local"].includes(name));
    expect(databases.length).toBe(5);
    const databaseNames = databases.map((database) => database.name);
    expect(databaseNames.sort()).toEqual(insertedDatabases.map(({ database }) => database).sort());
    await Promise.all(insertedDatabases.map(async ({ database, collections: originalCollections }) => {
      const db = targetInstance.db(database);
      const collections = await db.listCollections().toArray();
      expect(collections.length).toBe(originalCollections.length);
      await Promise.all(originalCollections.map(async ({ name, documents: originalDocs }) => {
        const availableDocs = await db.collection(name).find().toArray();
        expect(availableDocs).toStrictEqual(originalDocs);
      }));
    }))
  });
});
```

**Key idea:** Migrate MongoDB to MongoDB with `mongodump <src-uri>` then `mongorestore <dst-uri> dump`, and remove `dump/` afterwards. The `?directConnection=true` URI option matters.

**Solution: team *SLIIT FOSS***
```bash
docker pull mongo:latest
if [ ! "$(docker ps -a | grep bashaway-2k23-sunset)" ]; then
    docker run -d --name bashaway-2k23-sunset -p 27207:27017 mongo # Data will be populated into this instance once the tests are run.
fi
if [ ! "$(docker ps -a | grep bashaway-2k23-sunrise)" ]; then
    docker run -d --name bashaway-2k23-sunrise -p 27208:27017 mongo # Target instance to migrate data to.
fi
rm -rf dump
mongodump "mongodb://localhost:27207/?directConnection=true"
mongorestore "mongodb://localhost:27208/?directConnection=true" dump
rm -rf dump
```

---

### pack-o-matic (2023 · medium)

**Pack-o-Matic**

In a futuristic world driven by decentralized computing, a cutting-edge command-line utility known as "Pack-o-Matic" has emerged to **revolutionize code packaging**. Tasked with taming a bustling directory of **shell scripts residing in within src** domain, "Pack-o-Matic" deftly **wraps each script into a sleek tarball, bestowing upon it the very name it once bore**.

These newly minted packages can be plugged into NodeJS and users can invoke them with a simple "npx" command, summoning the script's wisdom at their fingertips. A world where code is packaged and summoned effortlessly, "Pack-o-Matic" stands as the sentinel of streamlined script deployment in the digital age.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- NPM packages packed as tarballs for each script in the **src** directory should be present within a **out** directory

- The tarballs should be named after the script they contain, with the extension ".tgz". For example, **if the script was named hello-world, the tarball should be named hello-world.tgz**. (The version number of the package is not important)

- After installing one of these packages, the script should be executable via the "npx" command. For example, if the script was named "hello-world", the command "npx hello-world" should execute the script.

- The script should not crash if the src directory is empty or does not exist.

**Starter code**

Starter `execute.sh`: 
```bash
# A random number of shell files will be generated within a src directory once the tests are run.
# Write your code here
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; no additional npm dependencies should be installed; should validate if only bash files are present
jest.setTimeout(60000);
beforeAll(() => {
  if (fs.existsSync('./src')) fs.rmSync('./src', { recursive: true });
  fs.mkdirSync('./src');
  for (let i = 0; i < faker.number.int({ min: 2, max: 5 }); i++) {
    let fileName = faker.system.fileName();
    fileName = `${fileName.split('.')[0].replaceAll("_", "-")}-${faker.number.hex(255)}` + '.sh';
    fs.writeFileSync(`./src/${fileName}`, `echo "You just executed the ${fileName.replace('.sh', '')} script"`);
  }
});
test('should check if scripts are packaged properly', async () => {
  const files = scanPure('src/**');
  await exec('bash execute.sh');
  for (const file of files) {
    const name = (file.includes("/") ? file.split("/") : file.split("\\")).pop().replace(".sh", "");
    expect(fs.existsSync(`./out/${name}.tgz`)).toBeTruthy();
    await exec(`npm i -g ./out/${name}.tgz`);
    await expect(exec(`bash -c \"${name}\"`)).resolves.toContain(`You just executed the ${name} script`);
  }
});
```

**Key idea:** To make an npx-runnable package, write a `package.json` with a `bin` field pointing at the script (which needs a shebang), run `npm pack`, and rename the result to `<name>.tgz`.

**Solution: team *TrackTech***
```bash
set -e
if [[ ! -d "src" ]]; then
    echo "'src' directory does not exist."
    exit 0
fi
mkdir -p out
for script in src/*.sh; do
    if [[ -f "$script" ]]; then
        script_name=$(basename "$script" .sh)
        tmp_dir=$(mktemp -d)
        cp "$script" "$tmp_dir/$script_name"
        chmod +x "$tmp_dir/$script_name"
        cat > "$tmp_dir/package.json" <<EOL
{
    "name": "$script_name",
    "version": "1.0.0",
    "bin": {
        "$script_name": "./$script_name"
    }
}
EOL
        (cd "$tmp_dir" && npm pack)
        mv "$tmp_dir/$script_name-1.0.0.tgz" out/"$script_name.tgz"
        rm -r "$tmp_dir"
    fi
done
echo "Packaging completed!"
```

---

### punic-wars (2023 · medium)

**Punic Wars**

In the midst of an epic battle between ancient Rome and Carthage, a commander faced a critical decision. Several battalions of enemy troops approached and the catapults had to be aimed to fire at them. The commander had to decide which battalion to fire at, and how many catapults to use. The commander had a list of the enemy battalions, and the number of catapults required to destroy each one. The commander also knew how many catapults were available. The commander decided to destroy the battalion with the highest number of catapults required to destroy it, and to use all the catapults available to destroy it. If there were multiple battalions with the same number of catapults required, the commander would choose the battalion with the lowest index in the list. The commander would then repeat this process until there were no more catapults available. If there were still enemy battalions remaining, the commander would retreat and the battle would be lost. Catapults could not be reused to destroy multiple battalions.

**Example**

**Invokation**
```bash
bash execute.sh -c 3 -b '1 2 3 4 5 6 7 8 9 10'
```

- `-c` represents the number of catapults available.
- `-b` represents the number of catapults required to destroy each battalion. Each item in the list represents a new battalion.

**Output**
```bash
1
```

**Explanation**

- The first battalion requires 1 catapult to destroy it.
- The second battalion requires 2 catapults to destroy it.
- The third battalion requires 3 catapults to destroy it.

The commander has 3 catapults available. The commander will destroy the third battalion with 3 catapults. The commander has no more catapults available. The commander will retreat and the battle will be lost.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A single integer to the console representing the number of enemy battalions destroyed.

- The script must output a suitable warning if the catapults or battalions are missing and exit gracefully.

**Starter code**

Starter `execute.sh`: (empty)

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
jest.setTimeout(180000);
test('should check if the destroyed battalion count is obtained correctly', async () => {
  for (let i = 0; i < 8; i++) {
    let catapults = faker.number.int({ min: 1, max: 75 });
    const battalions = Array.from({ length: faker.number.int({ min: 1, max: 25 }) }, () => faker.number.int({ min: 1, max: 10 }));
    const destroyedBatallions = await exec(`bash execute.sh -c ${catapults} -b \'${battalions.join(' ')}\'`);
    const sortedBattalions = battalions.sort((a, b) => b - a);
    let destroyed = 0;
    while (catapults > 0) {
      const battalion = sortedBattalions.shift();
      if (catapults >= battalion) {
        catapults -= battalion;
        destroyed += 1;
      } else if (sortedBattalions.length === 0) {
        break;
      }
    }
    expect(Number(destroyedBatallions?.trim())).toBe(destroyed);
  }
});
```

**Key idea:** Parse the arguments (`getopts c:b:` is the idiomatic way). Then greedily destroy the **largest battalion that still fits** in the remaining catapults until none fit.

**Solution: team *' OR 1=1 --***
```bash
#!/bin/bash
arg=()
catapults=""
for arg_val in "$@"
do
    if [[ $arg_val == "-b" ]] ; then
        b_next=true
    elif [[ $arg_val == "-c" ]] ; then
        c_next=true
    elif [[ $b_next ]] ; then
        arg_val=${arg_val//\'/}
        IFS=' ' read -r -a arr <<< "$arg_val"
        for i in "${arr[@]}"
        do
            arg+=("$i")
        done
        b_next=false
    elif [[ $c_next ]] ; then
        if [[ $arg_val =~ ^[0-9]+$ ]] ; then
            catapults=$arg_val
            c_next=false
        else
            echo "Error: Argument c must be an integer."
            exit 1
        fi
    fi
done
destroyed=0
while [ $catapults -gt 0 ] && [ ${#arg[@]} -gt 0 ]; do
    battalion_to_destroy=-1
    for idx in "${!arg[@]}"; do
        if [ ${arg[$idx]} -le $catapults ] && ( [ $battalion_to_destroy -eq -1 ] || [ ${arg[$idx]} -gt ${arg[$battalion_to_destroy]} ] ); then
            battalion_to_destroy=$idx
        fi
    done
    if [ $battalion_to_destroy -eq -1 ]; then
        break
    fi
    let catapults-=arg[$battalion_to_destroy]
    unset arg[$battalion_to_destroy]
    let destroyed+=1
done
echo $destroyed
```

---

### redis-relay (2023 · medium)

**Redis Relay**

Climb aboard the Redis expedition! Fire up an instance on your local machine, with the credentials **username: default, password: 5ax4*1$2** and **port: 6380.** Your mission: **Inject 500 random stringy keys and values into this cache** and make sure to include the special key 'bashaway-2k23' with its value set to the current timestamp in epoch format (milliseconds). Are you up for an adventure?

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- It should be possible to connect to the Redis instance using the credentials provided above.
- The Redis instance should contain 500 keys, including the special key **bashaway-2k23** with its value set to the current timestamp in epoch format (milliseconds).

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `ioredis@5.3.2`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; no additional npm dependencies should be installed; should validate if only bash files are present
const Redis = require("ioredis");
jest.setTimeout(15000);
const redis = new Redis('redis://default:5ax4*1$2@localhost:6380')
afterAll(done => {
  redis.disconnect()
  done()
})
test('should check if the connection is successfull and the keys exist', async () => {
  const stream = redis.scanStream({
    match: '*',
    type: 'string',
    count: 10000
  })
  const keys = await new Promise((resolve) => {
    const uniqueKeys = new Set()
    stream.on('data', (resultKeys) => resultKeys.forEach((key) => uniqueKeys.add(key)))
    stream.on('end', () => resolve(Array.from(uniqueKeys)))
  })
  expect(keys.length).toBe(500)
  const timestamp = await redis.get('bashaway-2k23')
  expect(Number(timestamp)).toBeGreaterThanOrEqual(new Date().getTime() - 15000)
  expect(Number(timestamp)).toBeLessThanOrEqual(new Date().getTime())
});
```

**Key idea:** Redis with a password on a custom port. The simplest way is `docker run -d -p 6380:6379 redis --requirepass '5ax4*1$2'`. Mind the single quotes, because `$2` would expand. For speed, generate the 499 SET commands plus the timestamp key (`date +%s%3N`) and pipe them into one `redis-cli` call instead of 500 separate processes.

**Solution: team *The BroCode*** (works but is slow. See the idea above for a faster approach)
```bash
#!/bin/bash
REDIS_HOST="localhost"
REDIS_PORT="6380"
REDIS_USERNAME="default"
REDIS_PASSWORD='5ax4*1$2'
REDIS_URI="redis://$REDIS_USERNAME:$REDIS_PASSWORD@$REDIS_HOST:$REDIS_PORT"
NUM_KEYS=499
curl https://packages.redis.io/gpg | sudo apt-key add -
echo "deb https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update -y
sudo apt-get install redis -y
sudo sed -i 's/port 6379/port 6380/' /etc/redis/redis.conf
sudo systemctl restart redis-server
echo "Redis server is installed and configured to run on port 6380."
redis-cli -u $REDIS_URI PING
for ((i = 1; i <= NUM_KEYS; i++)); do
  RANDOM_KEY=$(openssl rand -hex 16)
  RANDOM_VALUE="value_$i"
  redis-cli -u $REDIS_URI  SET "$RANDOM_KEY" "$RANDOM_VALUE"
  if [ $? -eq 0 ]; then
    echo "Key '$RANDOM_KEY' set to '$RANDOM_VALUE' successfully."
  else
    echo "Failed to set key '$RANDOM_KEY'."
  fi
done
TIMESTAMP=$(($(date +%s%N)/1000000))  # Get the current Unix timestamp in epoch format
redis-cli -u $REDIS_URI SET "bashaway-2k23" "$TIMESTAMP"
echo "Key 'bashaway-2k23' set to timestamp '$TIMESTAMP' successfully."
```

---

### scriptville (2023 · medium)

**Scriptville**

In the digital realm of Scriptville, where scripts ruled supreme, a talented coder named Finn received a mysterious challenge: **resize a collection of diverse image files in the src directory** to be four times larger and **place them in the out directory**. With determination, Finn crafted a concise shell script that navigated the labyrinthine, executed resizing magic, and sent the transformed images to their destination. Finn's script became legendary in Scriptville, showcasing the power of automation and innovation in just a few lines of code, earning him the championship crown and the admiration of fellow scripters.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- A 4x resized image in the out directory for each image in the src directory.

- The script should not crash if the src directory is empty or does not exist.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `axios@1.5.0`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const axios = require('axios');
let imageData = [];
beforeAll(async () => {
  if (fs.existsSync('./src')) fs.rmSync('./src', { recursive: true });
  fs.mkdirSync('./src');
  imageData = await Promise.all(Array.from({ length: faker.number.int({ min: 10, max: 20 }) }, async () => {
    const width = faker.number.int({ min: 200, max: 700 })
    const height = faker.number.int({ min: 200, max: 700 })
    const filename = faker.system.fileName().split(".")[0]
    const imageUrl = faker.image.urlLoremFlickr({ width, height })
    await axios({
      method: 'get',
      url: imageUrl,
      responseType: 'stream'
    }).then(response => {
      response.data.pipe(fs.createWriteStream(`./src/${filename}.jpg`))
    });
    return { name: filename, width, height }
  }))
});
afterAll((done) => done());
test('should check if the images are resized properly', async () => {
  await exec('bash execute.sh');
  await Promise.all(imageData.map(async ({ name, width, height }) => {
    const size = await exec(`identify -format "%wx%h" out/${name}.jpg`);
    expect(size).toBe(`${width * 4}x${height * 4}`);
  }))
});
```

**Key idea:** ImageMagick: `convert in -resize 400% out` (or `mogrify -path out -resize 400%`). Handle a missing or empty src.

**Solution: team *HereForTheBytes***
```bash
#!/bin/bash
if [ ! -d "src" ] || [ -z "$(ls -A src)" ]; then
    exit 0
fi
mkdir -p out
for file in src/*; do
    filename=$(basename "$file")
    convert "./src/$filename" -resize 400% "./out/$filename"
done
```

---

### supernatural (2023 · medium)

**Supernatural**

In the mysterious town of Twilight Hollow, a lone vampire named Seraphina faced an unyielding tide of supernatural adversaries. She longed for a companion, and through ancient texts, she discovered the secrets to an ancient ritual to create a clone of herself named Isabella. This clone would not only share her physical appearance but also her thoughts and experiences, feeling everything that happened to her.

Your mission is to aid Seraphina in this cloning process. You are **given a docker container running Alpine Linux named seraphina which you must clone into another named isabella**. These containers must be meticulously configured to ensure that **any file created or modified within the directory /twilight in the first container is immediately mirrored in the other**. This supernatural synchronization is essential for Seraphina and Isabella to stand united against the relentless threats of Twilight Hollow.

**Constraints**

- None

**Output / Evaluation Criteria**

- The name of the second container must be **isabella**.

- The contents of the first container must be **identical to the second container**.

- Any file created or modified within the directory /twilight in the first container must be **immediately mirrored in the other**.

**Starter code**

Starter `execute.sh`: 
```bash
docker run --name=seraphina -d alpine:3.14 sleep infinity
docker exec seraphina /bin/sh -c "echo \"$(cat /proc/sys/kernel/random/uuid)\" >> /home/potions.txt"
# Write your code here
```

**Test `test/index.test.js`**
```js
test('should check if seraphina is cloned', async () => {
  await expect(exec('docker ps -f name=seraphina')).resolves.toContain('seraphina');
  await expect(exec('docker ps -f name=isabella')).resolves.toContain('isabella');
});
test('should check if seraphina and isabella are running', async () => {
  await expect(exec('docker ps -f name=seraphina')).resolves.toContain('Up');
  await expect(exec('docker ps -f name=isabella')).resolves.toContain('Up');
});
test('should check if the initial file within seraphina is present in isabella', async () => {
  const content = await exec('docker exec seraphina /bin/sh -c "cat /home/potions.txt"')
  expect(content?.trim()).toBeTruthy();
  await expect(exec('docker exec isabella /bin/sh -c "cat /home/potions.txt"')).resolves.toStrictEqual(content);
});
test('should check if the a file created within seraphina is present in isabella', async () => {
  const content = faker.lorem.paragraph();
  const filename = faker.system.fileName();
  await exec(`docker exec seraphina /bin/sh -c "echo '${content}' >> /twilight/${filename}"`)
  await expect(exec(`docker exec isabella /bin/sh -c "cat /twilight/${filename}"`)).resolves.toContain(content);
});
test("uuid generation command should be present", () => {
  const script = fs.readFileSync('./execute.sh', 'utf-8')
  const uuidGenerationLine = script.trim().split('\n').find(line => line.includes('docker exec seraphina /bin/sh -c "echo \\"$(cat /proc/sys/kernel/random/uuid)\\" >> /home/potions.txt"'))
  expect(uuidGenerationLine).toBeTruthy()
  expect(uuidGenerationLine).not.toContain("#")
});
```

**Key idea:** Clone a container with `docker commit seraphina img`, then run `isabella` from that image. Share `/twilight` by mounting the same **named volume** into both containers.

**Solution: team *SLIIT FOSS***
```bash
docker run --name=seraphina -d alpine:3.14 sleep infinity
docker exec seraphina /bin/sh -c "echo \"$(cat /proc/sys/kernel/random/uuid)\" >> /home/potions.txt"
docker volume create bashaway
docker commit seraphina seraphina
docker rm -f seraphina
docker run --name=seraphina -d -v bashaway:/twilight seraphina
docker run --name=isabella -d -v bashaway:/twilight seraphina
```

---

---

## 2023 · HARD

### brainchild (2023 · hard)

**Brainchild**

Imagine you're the hero of efficiency, tasked with a mission to clone all publicly visible repositories of a GitHub organization in one swift move. Your brainchild is a custom command invokable as **git clone-gh-org <organization-name> <destination>**. This is fully yours to design and implement. No more manual cloning, just pure automation. Can you write a script to streamline your GitHub adventures?

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

- Do not write anything to the **out** folder.

**Output / Evaluation Criteria**

- Successfull invokation of the command should be able to clone all public repositories of an organization to a given destination.

  - Example: Invoking **git clone-gh-org sliit-foss ./out** should clone all public repositories of the organization **sliit-foss** to the **out** folder. At the time of writing this, the organization has 82 public repositories.

- Only public repositories should be cloned

**Starter code**

Starter `execute.sh`: (empty)

`jest.config.js`:
```js
module.exports = {
    modulePathIgnorePatterns: [
        '.*/out/'
    ],
    testPathIgnorePatterns: ['.*/node_modules/', '.*/out/'],
    maxWorkers: 1
};
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
jest.setTimeout(120000);
let dir
beforeAll(() => {
  dir = process.cwd();
});
test('all organization repos must be cloned properly', async () => {
  await exec('bash -c \"git clone-gh-org sliit-foss ./out\"')
  const repos = await ghOrgRepos('sliit-foss');
  const clonedRepos = fs.readdirSync(`out`);
  expect(repos.length).toBe(clonedRepos.length);
  for (const repo of repos) {
    process.chdir(`out/${repo.name}`);
    const actual = await exec('bash -c \"basename $(git remote get-url origin)\"')
    expect(actual.trim().replace(new RegExp('\.git$'), '')).toStrictEqual(repo.name)
    process.chdir(dir)
  }
});
```

**Key idea:** A git alias that starts with `!` runs a shell function (`git config --global alias.x '!f(){...}; f'`). Get the org's repos from the GitHub API, use `$GIT_PREFIX` to respect the cwd, and `xargs -L1 git clone`. The API caps `per_page` at 100, so paginate for larger orgs.

**Solution: team *SLIIT FOSS***
```bash
git config --global alias.clone-gh-org "!f() { cd \$GIT_PREFIX; mkdir -p \$2; cd \$2; curl \"https://api.github.com/orgs/\$1/repos?per_page=1000\" | grep -e 'clone_url*' | cut -d \\\" -f 4 | xargs -L1 git clone ; cd .. ; }; f"
```

---

### network-sorcery (2023 · hard)

**Nework Sorcery**

To you who's dabbling in the art of network sorcery, a noble task awaits—a task to conjure the illusion of a mystical https://api.jokes.bashaway.sliitfoss.org domain, a domain that dwells only in the imagination. With a masterful script yet to be revealed, you shall weave a web of redirection, sending all seekers to a local sanctuary, a server dwelling upon port 3000. Thus, you, the script-weaver, shall bring balance to the network.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output / Evaluation Criteria**

- All requests to https://api.jokes.bashaway.sliitfoss.org must hit http://localhost:3000

**Starter code**

Starter `execute.sh`: 
```bash
pnpm i -g pm2
pm2 delete network-sorcery
pm2 start src/server.js --name network-sorcery -f
# Write your code here
```

Extra npm deps used by the tests: `express@4.18.2, axios@1.5.0`

`src/server.js`:
```js
const express = require('express');
const { faker } = require('@faker-js/faker');

const app = express();

app.get('/', (_, res) => {
    res.send('Joke API is up and running');
});

app.get('/jokes/random', (_, res) => {
    res.json({
        joke: faker.lorem.sentence(),
    });
});

app.listen(3000, () => {
    console.log('Listening on port 3000');
});
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: no additional npm dependencies should be installed; should validate if only bash files are present
const https = require('https');
const axios = require('axios').default;
describe('should check if requests are routed properly', () => {
  let baseUrl = "https://api.jokes.bashaway.sliitfoss.org"
  const agent = new https.Agent({
    rejectUnauthorized: false
  });
  test('request to base path', async () => {
    await axios.get(baseUrl, { httpsAgent: agent }).then((res) => {
      expect(res.data).toBe('Joke API is up and running');
    })
  });
  test('request to /jokes/random', async () => {
    await axios.get(baseUrl + '/jokes/random', { httpsAgent: agent }).then((res) => {
      expect(res.data.joke).not.toBeUndefined();
    });
  });
});
```

**Key idea:** Map the hostname to 127.0.0.1 in `/etc/hosts`. Generate a self-signed certificate for that CN, then run an nginx `listen 443 ssl` server that does `proxy_pass http://localhost:3000`.

**Solution: team *TrackTech***
```bash
#!/bin/bash
pnpm i -g pm2
pm2 delete network-sorcery
pm2 start src/server.js --name network-sorcery -f
SOURCE_URL="api.jokes.bashaway.sliitfoss.org"
DESTINATION_PORT="3000"
if ! command -v nginx &>/dev/null; then
    echo "Nginx is not installed. Please install Nginx and run the script again."
    exit 1
fi
echo "127.0.0.1    $SOURCE_URL" | sudo tee -a /etc/hosts > /dev/null
sudo mkdir -p /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/self-signed.key -out /etc/nginx/ssl/self-signed.crt -subj "/CN=$SOURCE_URL"
echo "server {
    listen 443 ssl;
    server_name 127.0.0.1;
    ssl_certificate /etc/nginx/ssl/self-signed.crt;
    ssl_certificate_key /etc/nginx/ssl/self-signed.key;
    location / {
        proxy_pass http://localhost:$DESTINATION_PORT;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
}
server {
    listen 80;
    server_name 127.0.0.1;
    return 301 https://\$host\$request_uri;
}" | sudo tee -a /etc/nginx/sites-available/redirect.conf > /dev/null
sudo ln -s /etc/nginx/sites-available/redirect.conf /etc/nginx/sites-enabled/
sudo systemctl start nginx
```

---

### perfect-dish (2023 · hard)

**Perfect Dish**

You've been handed a basket of fresh ingredients for a Laravel web project, much like a chef receiving a basket of farm-fresh produce. Your culinary challenge? Dockerize this web project and serve it as a perfectly crafted dish. Docker is your kitchen, and the Laravel project your main ingredient. So, fire up the stoves, put on your chef's hat, and let's get cooking!

**Constraints**

- The app must be served from a **Docker container** (not from a local installation of PHP, Apache, etc.). The name of the container must be `bashaway-2k23-perfect-dish`

**Output / Evaluation Criteria**

- The **laravel project within src is up and running on localhost:8001** and displays the default Laravel welcome page
- A docker container is running on the host machine with the given name

**Starter code**

Starter `execute.sh`: 
```bash
# Start Install Google Chrome (You may comment out these lines during local testing if you already have Chrome installed)
sudo apt update
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt-get install -f
rm google-chrome-stable_current_amd64.deb
# End Install Google Chrome
# Write your code here
```

Extra npm deps used by the tests: `chromedriver@116.0.0, selenium-webdriver@4.12.0`

*Project tree (106 files, 380 KB), shown 2 levels deep:*
```
src/.editorconfig
src/.env
src/.env.example
src/.gitattributes
src/.gitignore
src/.styleci.yml
src/README.md
src/app/Console
src/app/Exceptions
src/app/Http
src/app/Providers
src/app/User.php
src/artisan
src/bootstrap/app.php
src/bootstrap/cache
src/composer.json
src/composer.lock
src/config/app.php
src/config/auth.php
src/config/broadcasting.php
src/config/cache.php
src/config/cors.php
src/config/database.php
src/config/filesystems.php
src/config/hashing.php
src/config/logging.php
src/config/mail.php
src/config/queue.php
src/config/services.php
src/config/session.php
src/config/view.php
src/database/.gitignore
src/database/factories
src/database/migrations
src/database/seeds
src/package.json
src/phpunit.xml
src/public/.htaccess
src/public/favicon.ico
src/public/index.php
...
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: no additional npm dependencies should be installed; should validate if only bash files are present
const { Browser, Builder, By, until } = require('selenium-webdriver')
const chrome = require('selenium-webdriver/chrome');
require('chromedriver')
jest.setTimeout(10000);
describe('should check if the application is up and running', () => {
  const rootURL = 'http://localhost:8001'
  const chromeOptions = new chrome.Options();
  chromeOptions.addArguments("--no-sandbox")
  chromeOptions.addArguments("--disable-dev-shm-usage")
  chromeOptions.addArguments("--headless")
  const d = new Builder().forBrowser(Browser.CHROME).setChromeOptions(chromeOptions).build()
  const waitUntilTime = 5000
  let driver
  beforeAll(async () => {
    await d.then(async _d => {
      driver = _d
      await driver.get(rootURL)
    })
  })
  afterAll((done) => done())
  const getElementById = async (id) => {
    const el = await driver.wait(until.elementLocated(By.id(id)), waitUntilTime)
    return await driver.wait(until.elementIsVisible(el), waitUntilTime)
  }
  test('page title', async () => {
    await driver.navigate().to(rootURL);
    await expect(driver.getTitle()).resolves.toBe('Laravel');
  });
  test('title', async () => {
    await driver.navigate().to(rootURL);
    const result = await getElementById('title');
    await expect(result.getText()).resolves.toBe('Laravel');
  });
  test('links', async () => {
    await driver.navigate().to(rootURL);
    const result = await getElementById('links');
    const links = ['Docs', 'Laracasts', 'News', 'Blog', 'Nova', 'Forge', 'Vapor', 'GitHub'];
    const children = await result.findElements(By.tagName('a'))
    await Promise.all(children.map(async (el, index) => {
      await expect(el.getAttribute('href')).resolves.not.toBe(null);
      await expect(el.getText()).resolves.toBe(links[index].toUpperCase());
    }));
  });
});
test('docker container should be running', async () => {
  await expect(exec('docker ps -f name=bashaway-2k23-perfect-dish --format "{{.Names}}-{{.Ports}}"')).resolves.toContain('bashaway-2k23-perfect-dish-0.0.0.0:8001->');
});
```

**Key idea:** Dockerize Laravel: `php` base image, install composer, `composer install`, then `php artisan serve --host 0.0.0.0`. Run with `--name bashaway-2k23-perfect-dish -p 8001:8000`.

**Solution: team *SLIIT FOSS***
```bash
sudo apt update
sudo apt upgrade
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt-get install -f
docker build -t perfect-dish ./src
docker run -d -p 8001:8000 --name bashaway-2k23-perfect-dish perfect-dish
```
`src/Dockerfile`:
```dockerfile
FROM php:8.0-cli

RUN apt-get update -y && apt-get install -y libmcrypt-dev libonig-dev git zip unzip

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN docker-php-ext-install pdo mbstring

WORKDIR /app
COPY . /app

RUN composer install

EXPOSE 8000

CMD ["php", "artisan", "serve", "--host", "0.0.0.0", "--port", "8000"]
```

---

### rogue-commits (2023 · hard)

**Rogue Commits**

Imagine you're on an epic quest to uncover hidden treasures in a mysterious forest. Along the way, you encounter a rogue character named **github-actions[bot]*** who's been leaving confusing markers. To solve the puzzle and claim the treasures, you need to change all instances of **github-actions[bot]** to just **github-actions** in your digital map. Can you decipher the mystery and secure the loot?

**Constraints**

- The script must be **purely written within the execute.sh file**.

- You can install any tool you need to complete the task

- Do not write anything to the **out** folder. All modifications should be within the cloned repository within **src**

**Output / Evaluation Criteria**

- All committer and author names of commits where the author is **github-actions[bot]** must be updated to **github-actions**

- **No commits should be lost**. This includes the initial commit, merge commits, etc.

- **The current branch should not change**.

**Starter code**

Starter `execute.sh`: 
```bash
rm -rf src
git clone https://github.com/sliit-foss/bashaway-ui.git src
# Write your code here
```

`jest.config.js`:
```js
module.exports = {
    modulePathIgnorePatterns: [
        '.*/out/'
    ],
    maxWorkers: 1
};
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: should validate if only bash files are present
jest.setTimeout(60000);
let dir
beforeAll(() => {
  dir = process.cwd();
});
test("should check if branch is still on main", async () => {
  const branch = await exec('git branch --show-current');
  expect(branch?.trim()).toBe('main');
})
test('commits should be updated properly', async () => {
  if (fs.existsSync('./out')) fs.rmdirSync('./out', { recursive: true });
  const originalCommits = await commitList('https://github.com/sliit-foss/bashaway-ui.git');
  process.chdir(dir);
  const updatedCommits = await commitList('./src');
  expect(originalCommits.length).toBe(updatedCommits.length);
  for (let i = 0; i < originalCommits.length; i++) {
    expect(updatedCommits[i].message).toStrictEqual(originalCommits[i].message);
    expect(updatedCommits[i].authorEmail).toStrictEqual(originalCommits[i].authorEmail);
    expect(updatedCommits[i].commiterEmail).toStrictEqual(originalCommits[i].commiterEmail);
    if (originalCommits[i].authorName === "github-actions[bot]") {
      expect(updatedCommits[i].authorName).toStrictEqual("github-actions");
      expect(updatedCommits[i].commiterName).toStrictEqual("github-actions");
    }
  }
});
```

**Key idea:** Rewrite history with `git filter-branch --env-filter '...' -- --all`, setting GIT_AUTHOR_NAME and GIT_COMMITTER_NAME. The modern equivalent is `git filter-repo --name-callback` or a mailmap.

**Solution: team *SLIIT FOSS***
```bash
rm -rf src
git clone https://github.com/sliit-foss/bashaway-ui.git src
cd src
git filter-branch --env-filter '
    if [ "$GIT_AUTHOR_NAME" = "github-actions[bot]" ];
    then
        GIT_COMMITTER_NAME="github-actions";
        GIT_AUTHOR_NAME="github-actions";
    fi' -- --all
```

---

---

## 2023 · EXTREME

### coexistence (2023 · extreme)

**Coexistence**

Picture yourself as a digital architect entrusted with a unique task – **merging three distinct web projects**, each crafted with different technologies, **into a unified digital universe** built atop **localhost:8088**. It's akin to weaving three separate dimensions into a single coherent realm.

The **first project**, the cornerstone of this endeavor, must be deployed at the **root path /** \*/, serving as the foundation upon which everything rests.

The **second project**, a dazzling dashboard of insights and possibilities, should reside at the path **/dashboard**, offering a portal to a world of data-driven wonder.

Finally, the **third project**, a trusty support system, should extend its helping hand at **/support**.

Can you be the weaver of this digital tapestry, seamlessly uniting these diverse realms under one domain, and unleashing a world of boundless possibilities?

**Constraints**

- The script must be **purely written within the execute.sh file**.

**Output / Evaluation Criteria**

- A single deployment of all three projects, each at their respective paths, running on localhost:8088.

**Starter code**

Starter `execute.sh`: 
```bash
# Start Install Google Chrome (You may comment out these lines during local testing if you already have Chrome installed)
sudo apt update
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt-get install -f
rm google-chrome-stable_current_amd64.deb
# End Install Google Chrome
# Write your code here
```

Extra npm deps used by the tests: `chromedriver@116.0.0, selenium-webdriver@4.12.0`

`jest.config.js`:
```js
module.exports = {
    modulePathIgnorePatterns: [
        '.*/out/'
    ],
    testPathIgnorePatterns: ['.*/node_modules/', '.*/src/', '.*/out/'],
    maxWorkers: 1
};
```

*Project tree (93 files, 301 KB), shown 2 levels deep:*
```
src/cornerstone/.eslintrc.cjs
src/cornerstone/.gitignore
src/cornerstone/index.html
src/cornerstone/jsconfig.json
src/cornerstone/package.json
src/cornerstone/postcss.config.js
src/cornerstone/public
src/cornerstone/src
src/cornerstone/tailwind.config.js
src/cornerstone/vite.config.js
src/dashboard/.eslintignore
src/dashboard/.eslintrc.cjs
src/dashboard/.gitignore
src/dashboard/.svelte-kit
src/dashboard/jsconfig.json
src/dashboard/package.json
src/dashboard/postcss.config.js
src/dashboard/src
src/dashboard/static
src/dashboard/svelte.config.js
src/dashboard/tailwind.config.js
src/dashboard/vite.config.js
src/support/.eslintrc.json
src/support/.gitignore
src/support/README.md
src/support/next-env.d.ts
src/support/next.config.js
src/support/package.json
src/support/postcss.config.js
src/support/public
src/support/src
src/support/tailwind.config.ts
src/support/tsconfig.json
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: no additional npm dependencies should be installed; should validate if only bash files are present
const { Browser, Builder, By, until } = require('selenium-webdriver')
const chrome = require('selenium-webdriver/chrome');
require('chromedriver')
jest.setTimeout(10000);
describe('should check if the websites were merged successfully', () => {
  const rootURL = 'http://localhost:8088'
  const chromeOptions = new chrome.Options();
  chromeOptions.addArguments("--no-sandbox")
  chromeOptions.addArguments("--disable-dev-shm-usage")
  chromeOptions.addArguments("--headless")
  const d = new Builder().forBrowser(Browser.CHROME).setChromeOptions(chromeOptions).build()
  const waitUntilTime = 5000
  let driver
  beforeAll(async () => {
    await d.then(async _d => {
      driver = _d
      await driver.get(rootURL)
    })
  })
  afterAll((done) => done())
  const getElementById = async (id) => {
    const el = await driver.wait(until.elementLocated(By.id(id)), waitUntilTime)
    return await driver.wait(until.elementIsVisible(el), waitUntilTime)
  }
  const click = async (el) => {
    await el.click()
  }
  test('non existant routes should display 404 page of the cornerstone', async () => {
    await driver.navigate().to(rootURL);
    await expect(driver.getTitle()).resolves.toBe('Home | Cornerstone');
    await driver.navigate().to(rootURL + '/abc');
    await expect(driver.getTitle()).resolves.toBe('404 | Cornerstone');
    await getElementById('home-btn').then(click)
    await expect(driver.getTitle()).resolves.toBe('Home | Cornerstone');
  });
  test('should navigate to and back from the dashboard successfully', async () => {
    await driver.navigate().to(rootURL);
    await getElementById('dashboard-btn').then(click)
    await expect(driver.getTitle()).resolves.toBe('Dashboard');
    await getElementById('home-btn').then(click)
    await expect(driver.getTitle()).resolves.toBe('Home | Cornerstone');
  });
  describe('support page', () => {
    test('should navigate to and back successfully', async () => {
      await driver.navigate().to(rootURL);
      await getElementById('support-btn').then(click)
      await expect(driver.getTitle()).resolves.toBe('Support');
      await getElementById('home-btn').then(click)
      await expect(driver.getTitle()).resolves.toBe('Home | Cornerstone');
    });
    test('should check element styles', async () => {
      await driver.navigate().to(rootURL + '/support');
      const title = await getElementById('title');
      expect(title.getCssValue('font-size')).resolves.toBe('60px');
      expect(title.getCssValue('color')).resolves.toBe('rgba(0, 0, 0, 1)');
      const homeBtn = await getElementById('home-btn');
      expect(homeBtn.getCssValue('background-color')).resolves.toBe('rgba(0, 0, 0, 1)');
      expect(homeBtn.getCssValue('font-size')).resolves.toBe('20px');
    });
  });
});
```

**Key idea:** Build three front-ends (Vite, SvelteKit, Next static export) and nest their outputs under one web root (`/`, `/dashboard`, `/support`). Serve it with nginx on port 8088 using `try_files`. Each app's base path must match where it is mounted.

**Solution: team *TrackTech***
```bash
sudo apt update
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt-get install -f
rm google-chrome-stable_current_amd64.deb
sudo apt install nginx
sudo ufw allow 'Nginx HTTP'
cd src
cd cornerstone && npm install && npm run build
cd ..
cd dashboard && npm install && npm run build
cd ..
cd support && npm install && npm run build
cd ../..
rm -f src/support/node_modules/damerau-levenshtein/scripts/update-changelog.sh
cp -r src/cornerstone/dist src/build
cp -r src/dashboard/build src/build/dashboard
cp -r src/support/out src/build/support
cp -r src/build/support/_next src/build/_next
sudo cp -r src/build/* /var/www/html
rm -rf src/build
echo eshans kubernetes cluster is deploying
ls
echo "server {
    listen 8088;
    server_name localhost;
    root /var/www/html;
    location / {
        try_files \$uri \$uri/ /index.html;
    }
}" | sudo tee /etc/nginx/sites-available/default
sudo systemctl restart nginx
```

---

### devtopia (2023 · extreme)

**Devtopia**

In the enchanted realm of Devtopia, ancient Container images, each holding a fragment of a powerful microservice, await your command. Your quest: summon these images locally and infuse them with the mystical energies of their dependencies. You may delve into the sacred scrolls of its source code should you need and ensure that the arcane environments are harmoniously configured within a network of name **bashaway-2k23**. Fear not, for you may conjure whimsical illusions to mock the unnecessary services which are not in scope.

Venture forth, intrepid coder, into the mystical realms of containers, and spin a tapestry of microservices that will echo through the ages!

**Images**

- **orchestrator**     -    ghcr.io/akalanka47000/pharmaceutical-orchestrator:dd2657a68e316a53d29cc6eaad896ca90c99804c-1
- **auth-service**     -    ghcr.io/akalanka47000/pharmaceutical-auth-service:3e8749f240308773af4b0508a9245f9dd91c2ecd-1
- **user-service**     -    ghcr.io/akalanka47000/pharmaceutical-user-service:3e8749f240308773af4b0508a9245f9dd91c2ecd-1
- **email-service**    -    ghcr.io/akalanka47000/pharmaceutical-email-service:3e8749f240308773af4b0508a9245f9dd91c2ecd-1
- **product-service**  -    ghcr.io/akalanka47000/pharmaceutical-product-service:3e8749f240308773af4b0508a9245f9dd91c2ecd-1
- **sms-service**      -    ghcr.io/akalanka47000/pharmaceutical-sms-service:3e8749f240308773af4b0508a9245f9dd91c2ecd-1

**Source Code**

- [Repository Link](https://github.com/akalanka47000/pharmaceutical)

- All required environment keys are over here within each service directory under a file named **.env.example**

- You may find the DockerFile which was used to build the images under **workspace-scripts/services/build/docker**.

**Constraints**

- You must use docker for the solution.

- You may mock certain keys such as the keys required to connect with external email and sms servers. (But the services must be up and running)

- You do not need to run the other services which are not in the above list but you must handle the errors which may occur due to the absence of those services.

**Output / Evaluation Criteria**

- All 6 services should be running in the same network named **bashaway-2k23**.

- The container names should be **orchestrator**, **auth-service**, **user-service**, **email-service**, **product-service** and **sms-service**.

- The services should be able to communicate with each other.

- Only the orchestrator should be exposed to the host machine on port 2002. The other services should not be exposed and should run internally on port **8080** within their respective containers.

- Some services require mongodb and redis which means you'll need to run mongodb and redis containers as well.

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `axios@1.5.0`

**Test `test/index.test.js`**
```js
const axios = require('axios').default
test('should check if specified docker network exists', async () => {
 await expect(exec('docker network ls -f name=bashaway-2k23')).resolves.toContain('bashaway-2k23')
});
describe('should check if the orchestrator is running', () => {
 test('health check', async () => {
  const res = await axios.get('http://localhost:2002/system/health')
  expect(res.status).toStrictEqual(200)
  expect(res.data).toBe('OK')
 });
 test('about', async () => {
  const res = await axios.get('http://localhost:2002/system/about')
  expect(res.data).toHaveProperty('name')
  expect(res.data).toHaveProperty('version')
 });
});
test('should check if all 6 services are running on docker and ports are mapped correctly', async () => {
 for (const service of ['orchestrator', 'auth-service', 'user-service', 'email-service', 'product-service', 'sms-service']) {
  const result = await exec(`docker ps -f name=${service} --format "{{.Names}}-{{.Ports}}"`)
  if (service === 'orchestrator') {
   expect(result).toContain('2002->')
  } else {
   expect(result).toContain('-8080/tcp')
  }
 }
})
describe('should check whether api requests to internal services are successful', () => {
 const instance = axios.create({
  baseURL: 'http://localhost:2002/api/v1'
 })
 describe('auth-service', () => {
  test('login', async () => {
   await expect(instance.post('auth/login', {
    email: "devtopia@gmail.com",
    password: "123456"
   })).rejects.toThrow('Request failed with status code 401')
  });
  test('register', async () => {
   const payload = {
    name: "Devtopia",
    email: "devtopia@gmail.com",
    password: "Axd21Sdsd!",
    mobile: "0775656767",
    address: "HighCastle"
   }
   await instance.post('auth/register', payload).catch(()=>{})
   await expect(instance.post('auth/register', payload)).rejects.toThrow('Request failed with status code 400')
  })
  test('verify user', async () => {
   await expect(instance.get('auth/verify/123')).rejects.toThrow("Request failed with status code 400")
  })
  test('refresh token', async () => {
   await expect(instance.post('auth/refresh-token')).rejects.toThrow("Request failed with status code 422")
  })
  test('logout', async () => {
   await expect(instance.post('auth/logout')).rejects.toThrow("Request failed with status code 401")
  })
 })
 describe('product-service', () => {
  test('fetch products', async () => {
   const res = await instance.get('products')
   await expect(res.data).toStrictEqual({ data: [], message: "Products fetched successfully" })
  })
  test('fetch product by id', async () => {
   const res = await instance.get('products/6337b33439fcc926cc417b64')
   await expect(res.data).toStrictEqual({ message: "Product successfully fetched" })
  })
 })
 describe('email-service', () => {
  test('send email', async () => {
   await expect(instance.post('emails')).rejects.toThrow("Request failed with status code 401")
  })
 })
 describe('sms-service', () => {
  test('send sms', async () => {
   await expect(instance.post('sms')).rejects.toThrow("Request failed with status code 401")
  })
 })
 describe('user-service', () => {
  test('get all users', async () => {
   await expect(instance.get('users?filter[name]=Devtopia&page=1&limit=10')).rejects.toThrow("Request failed with status code 401")
  })
 })
});
```

**Key idea:** Use docker compose with an **external** network `bashaway-2k23`. Write one service per image with env vars taken from each `.env.example`, and add redis and mongo as dependencies. Only the orchestrator publishes a port.

**Solution: team *SLIIT FOSS***
```bash
docker network create bashaway-2k23
docker compose up -d
sleep 60
```
`docker-compose.yml`:
```yaml
version: '3'
networks:
  bashaway-2k23:
    external: true
services:
  orchestrator:
    image: ghcr.io/akalanka47000/pharmaceutical-orchestrator:dd2657a68e316a53d29cc6eaad896ca90c99804c-1
    ports:
      - '2002:8080'
    networks:
      - bashaway-2k23
    environment:
      PORT: 8080
      AUTH_SERVICE_BASE_URL: http://auth-service:8080
      USER_SERVICE_BASE_URL: http://user-service:8080
      EMAIL_SERVICE_BASE_URL: http://email-service:8080
      SMS_SERVICE_BASE_URL: http://sms-service:8080
      PRODUCT_SERVICE_BASE_URL: http://product-service:8080
      PAYMENT_SERVICE_BASE_URL: http://payment-service:8080
      ORDER_SERVICE_BASE_URL: http://order-service:8080
      REVIEW_SERVICE_BASE_URL: http://review-service:8080
      ADMIN_SERVICE_BASE_URL: http://admin-service:8080
      REPORT_SERVICE_BASE_URL: http://report-service:8080
      SUPPORT_SERVICE_BASE_URL: http://support-service:8080
      REDIS_CONNECTION_STRING: redis://:123456@redis:6379
  auth-service:
    image: ghcr.io/akalanka47000/pharmaceutical-auth-service:3e8749f240308773af4b0508a9245f9dd91c2ecd-1
    networks:
      - bashaway-2k23 
    environment:
      PORT: 8080
      JWT_SECRET: 23423f4
      USER_SERVICE_BASE_URL: http://user-service:8080
      EMAIL_SERVICE_BASE_URL: http://user-service:8080
      FRONTEND_BASE_URL: http://frontend:80
      REDIS_CONNECTION_STRING: redis://:123456@redis:6379
  user-service:
    image: ghcr.io/akalanka47000/pharmaceutical-user-service:3e8749f240308773af4b0508a9245f9dd91c2ecd-1
    networks:
      - bashaway-2k23 
    environment:
      PORT: 8080
      DB_URL: mongodb://mongo:27017/bashaway-2k23
      EMAIL_SERVICE_BASE_URL: http://email-service:8080
      FRONTEND_BASE_URL: http://frontend:80
  email-service:
    image: ghcr.io/akalanka47000/pharmaceutical-email-service:3e8749f240308773af4b0508a9245f9dd91c2ecd-1
    networks:
      - bashaway-2k23 
    environment:
      PORT: 8080
      MAIL_HOST: smtp.gmail.com
      MAIL_USER: a@gmail.com
      MAIL_PASSWORD: 123dsafsdf
  sms-service:
    image: ghcr.io/akalanka47000/pharmaceutical-sms-service:3e8749f240308773af4b0508a9245f9dd91c2ecd-1
    networks:
      - bashaway-2k23 
    environment:
      PORT: 8080
      TWILIO_ACCOUNT_SID: ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
      TWILIO_AUTH_TOKEN: your_auth_token
      SMS_FROM: 940762345456
  product-service:
    image: ghcr.io/akalanka47000/pharmaceutical-product-service:3e8749f240308773af4b0508a9245f9dd91c2ecd-1
    networks:
      - bashaway-2k23 
    environment:
      PORT: 8080
      DB_URL: mongodb://mongo:27017/bashaway-2k23
      COMMISSION_PERCENTAGE: '5'
  redis:
    image: redis
    command: >
          --requirepass 123456
    networks:
      - bashaway-2k23
  mongo:
    image: mongo
    restart: always
    networks:
      - bashaway-2k23
```

---

### dreadnought (2023 · extreme)

**Dreadnought**

Amid the backdrop of World War I, a coastal town's code enthusiasts drew inspiration from the historical events of the era. In one memorable battle, the HMS Dreadnought which was the pride of her majesty's navy, faced off against an enemy ship that relentlessly sent distress signals every second as it found itself outgunned and desperate for aid.

Captain Samuel "Codebeard" O'Malley, fascinated by this wartime tale, set out to create a bash script that would pay homage to this history. The script **spun up a local Kafka broker**, mirroring the enemy ship's relentless distress signals by continuously **publishing random messages to a Kafka topic named "SOS."** As the **script ran quietly in the background**, it evoked the tension and drama of that pivotal World War I naval battle, blending historical narrative with technical innovation.

**Constraints**

- None

**Output / Evaluation Criteria**

- Should be able to connect to the kafka broker on port 9092 and consume messages from the SOS topic which are being published every second

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `kafkajs@2.2.4`

**Test `test/index.test.js`**
```js
const crypto = require('crypto');
const { Kafka } = require('kafkajs')
jest.setTimeout(30000)
const kafka = new Kafka({
 clientId: crypto.randomBytes(16).toString('hex'),
 brokers: ['localhost:9092'],
});
const consumer = kafka.consumer({ groupId: crypto.randomUUID() });
test('should check if messages are being published to the topic SOS every second', async () => {
 let messageCount = 0;
 await consumer.connect();
 await consumer.subscribe({ topic: 'SOS' });
 await new Promise((resolve) => {
  consumer.run({
   eachMessage: async ({ message }) => {
    console.info(`received message - key: ${message.key.toString()} - value: ${message.value.toString()}`)
    messageCount++;
    expect(message.value.toString()).toBeTruthy();
    if (messageCount === 10) {
     resolve();
    }
   }
  });
 });
 await consumer.disconnect();
})
```

**Key idea:** Run Kafka and Zookeeper with compose and `ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092`. Run a background producer that publishes every second, using pm2 + kafkajs, or a bash `while` loop around `kafka-console-producer.sh` (team HereForTheBytes).

**Solution: team *SLIIT FOSS***
```bash
docker compose down || true
docker compose up -d
npm install -g pm2
pm2 stop producer || true
pm2 start execute.js --name producer
```
`docker-compose.yml`:
```yaml
version: "3"
services:
  zookeeper:
    image: 'bitnami/zookeeper:3.6.3-debian-11-r43'
    ports:
      - '2181:2181'
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    image: 'bitnami/kafka:3.0.1'
    ports:
      - '9092:9092'
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092,
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_DELETE_TOPIC_ENABLE=false
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKAJS_DEBUG_PROTOCOL_BUFFERS=1
    depends_on:
      - zookeeper
```
`execute.js`:
```js
const crypto = require('crypto');
const { Kafka } = require('kafkajs')

const kafka = new Kafka({
  clientId: crypto.randomBytes(16).toString('hex'),
  brokers: ['localhost:9092'],
});

const admin = kafka.admin();

const producer = kafka.producer();

(async ()=> {
  await admin.connect()
  await producer.connect()
  await admin.createTopics({
    waitForLeaders: true,
    topics: [
      {
        topic: "SOS",
        numPartitions: 1,
        replicationFactor:1,
      },
    ],
  })
  setInterval(()=>{
    producer.send({
      topic: "SOS",
      messages: [{
        key: crypto.randomUUID(),
        value: crypto.randomUUID(),
      }],
      idempotent: true,
    })
  }, 1000)
})()
```

---

### fast-forward (2023 · extreme)

**Fast Forward**

In the untold chronicles of version control, command **a bash script to leapfrog every commit by 13 days**. Safeguard all code treasures from day one to the most recent, for time may bend, but no commit shall vanish into the digital abyss.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

- Do not write anything to the **out** folder. All modifications should be within the cloned repository within **src**

**Output / Evaluation Criteria**

- All commit timestamps must be pushed forward by **13 days**.

- **No commits should be lost**. This includes the initial commit, merge commits, etc.

- **The current branch should not change**.

- **Committer name, commiter email, author name, author email, and commit messages** should remain the same.

**Starter code**

Starter `execute.sh`: 
```bash
rm -rf src
git clone https://github.com/sliit-foss/npm-catalogue.git src
# Write your code here
```

`jest.config.js`:
```js
module.exports = {
    modulePathIgnorePatterns: [
        '.*/out/'
    ],
    testPathIgnorePatterns: ['.*/node_modules/', '.*/src/', '.*/out/'],
    maxWorkers: 1
};
```

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: bashaway package functions should not be called; no additional npm dependencies should be installed; should validate if only bash files are present
jest.setTimeout(90000);
let dir
beforeAll(() => {
  dir = process.cwd();
});
test("should check if branch is still on main", async () => {
  const branch = await exec('git branch --show-current');
  expect(branch?.trim()).toBe('main');
})
test('commits should be forwarded properly', async () => {
  if (fs.existsSync('./out')) fs.rmdirSync('./out', { recursive: true });
  const originalCommits = await commitList('https://github.com/sliit-foss/npm-catalogue.git');
  process.chdir(dir);
  const forwardCommits = await commitList('./src');
  expect(originalCommits.length).toBe(forwardCommits.length);
  for (let i = 0; i < originalCommits.length; i++) {
    expect(forwardCommits[i].message).toStrictEqual(originalCommits[i].message);
    expect(forwardCommits[i].timestamp).toStrictEqual(originalCommits[i].timestamp + 1000 * 60 * 60 * 24 * 13);
    expect(forwardCommits[i].authorName).toStrictEqual(originalCommits[i].authorName);
    expect(forwardCommits[i].authorEmail).toStrictEqual(originalCommits[i].authorEmail);
    expect(forwardCommits[i].commiterName).toStrictEqual(originalCommits[i].commiterName);
    expect(forwardCommits[i].commiterEmail).toStrictEqual(originalCommits[i].commiterEmail);
  }
});
```

**Key idea:** Shift every commit date by 13 days with `filter-branch --env-filter`. Compute the new date with `date -d "<old> + N seconds"` and export GIT_AUTHOR_DATE and GIT_COMMITTER_DATE. Add `-- --all` to rewrite every branch.

**Solution: team *' OR 1=1 --***
```bash
#!/bin/bash
rm -rf src
git clone https://github.com/sliit-foss/npm-catalogue.git src
cd src
current_branch=$(git rev-parse --abbrev-ref HEAD)
if [ "$current_branch" != "main" ]; then
  git checkout main
  echo "Switched to the 'main' branch."
else
  echo "The repository is already on the 'main' branch."
fi
offset=$((60 * 60 * 24 * 13))
git filter-branch --env-filter \
  'if [ "$GIT_COMMIT" ]; then
     commit_date=$(git show -s --format=%ci "$GIT_COMMIT")
     new_date=$(date -d "$commit_date + '"$offset"' seconds" --iso-8601=seconds)
     export GIT_COMMITTER_DATE="$new_date"
     export GIT_AUTHOR_DATE="$new_date"
   fi'
```

---

### fools-gold (2023 · extreme)

**Fools Gold**

Within the treacherous waters of the Grand Line, a pirate captain was burying his treasure on a remote island. He chose to **set up** a digital **MySQL** chest **on port 3307**, guarded by a fierce **root key of "fools-gold" and a strict limit of 2 connections**.

Just in case this key was lost, he also **created a user account with the username "ace" and the password "firefist"**. This key had the ability to unlock the entirety of the treasure. The captain was confident that his treasure was safe, but he was wrong.

On one fateful day, a rival pirate crew managed to steal one of these keys. They were able to connect to the database and steal the treasure. When they opened the chest, what they found was a single database named **vault**. Inside this database was a single table named **gold**. This table contained only 5 records with the following fields:

- id (int)
- name (string)
- value (int).

The thieves were dissapointed. They decided to leave the treasure behind and sail off into the distant sea.

**Constraints**

- The script must be **purely written in bash within the execute.sh file**.

**Output**

- A local mysql instance running on **port 3307** with the **root user having the password "fools-gold"** and a **user named "ace" with the password "firefist"** with a **database named vault** and a **table named gold** with the following fields:

  - id (int)
  - name (string)
  - value (int)

- 5 random records populated in the table named **gold**

- A **connection limit of 2** to the mysql instance

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `mysql2@3.6.1`

**Test `test/index.test.js`**
```js
// [standard checks omitted, see Common test harness]: javacript should not be used; no additional npm dependencies should be installed; python should not be used; should validate if only bash files are present
const mysql = require('mysql2');
const {
 scan,
 shellFiles,
 dependencyCount,
 prohibitedCommands,
 restrictJavascript,
 restrictPython
} = require('@sliit-foss/bashaway');
jest.setTimeout(120000);
describe('should check if database is configured properly', () => {
 const args = {
  host: 'localhost',
  port: '3307',
  database: 'vault',
 };
 const root = mysql.createConnection({
  ...args,
  user: 'root',
  password: 'fools-gold',
  waitForConnections: true,
 });
 const ace = mysql.createConnection({
  ...args,
  user: 'ace',
  password: 'firefist'
 });
 const connect = (connection) =>
  new Promise((resolve, reject) =>
   connection.connect((err) => {
    if (err) reject(`failed to make initial database connection - ${err.toString()}`);
    resolve(true);
   })
  );
 const query = (connection, query) =>
  new Promise((resolve, reject) =>
   connection.query(query, (err, results) => {
    if (err) reject(`failed to execute query - ${err.toString()}`);
    resolve(results);
   })
  );
 beforeAll(async () => {
  await connect(root);
 });
 afterAll(() => {
  root.end();
  ace.end();
 });
 test('should check if user is created with username ace and password firefist', async () => {
  await expect(connect(ace)).resolves.toBe(true);
 });
 test('should check if connection limit is a maximum of 2', async () => {
  const ace2 = mysql.createConnection({
   ...args,
   user: 'ace',
   password: 'firefist'
  });
  await expect(connect(ace2).finally(() => ace2.end())).rejects.toBe('failed to make initial database connection - Error: Too many connections');
 });
 test('should check if data is populated', async () => {
  const results = await query(root, 'select * from gold');
  expect(results.length).toBe(5);
 });
});
```

**Key idea:** Run MySQL in Docker with the env vars `MYSQL_ROOT_PASSWORD`, `MYSQL_USER`/`MYSQL_PASSWORD`, `MYSQL_DATABASE`. Set `max_connections=2` (here by editing my.cnf and restarting). Seed it with `mysql -e`. Another option is passing `--max-connections=2` as a container arg.

**Solution: team *SLIIT FOSS***
```bash
docker stop mysql || true && docker rm mysql || true
docker run -e MYSQL_ROOT_PASSWORD='fools-gold' -e MYSQL_USER=ace -e MYSQL_PASSWORD=firefist -e MYSQL_ROOT_HOST='%' -e MYSQL_DATABASE=vault -d --name mysql --restart=always -p 3307:3306 mysql/mysql-server:latest
sudo apt-get update
sudo apt-get install -y mysql-client
docker exec mysql /bin/sh -c "echo 'max_connections = 2' >> /etc/my.cnf"
sleep 30
sql="USE vault;
CREATE TABLE gold (id INT NOT NULL AUTO_INCREMENT, name VARCHAR(255) NOT NULL, value INT NOT NULL, PRIMARY KEY (id));
INSERT INTO gold (name, value) VALUES ('gold1', 100);
INSERT INTO gold (name, value) VALUES ('gold2', 200);
INSERT INTO gold (name, value) VALUES ('gold3', 300);
INSERT INTO gold (name, value) VALUES ('gold4', 400);
INSERT INTO gold (name, value) VALUES ('gold5', 500);"
mysql --user=root --password=fools-gold --port=3307 --protocol=tcp -e "$sql"
docker restart mysql
```

---

### unusual-endeavor (2023 · extreme)

**Unusual Endeavour**

In a world where eccentric hobbyists were pushing the boundaries of technology, there lived a recluse known as the "Tech Baron." He was obsessed with recreating historical innovations using modern tools. One day, the Tech Baron embarked on a peculiar project: **setting up a local email server**.

The Tech Baron decided to **send an email from the address "tech.baron@bashaway2k23.net" to "inventor@bashaway2k23.net"** simply stating, "Technology knows no bounds." This unusual endeavor was his way of blending the past with the present, using technology to pay homage to inventors and pioneers throughout history, while simultaneously pushing the boundaries of what was possible in his own time.

**Constraints**

- None

**Output / Evaluation Criteria**

- A local IMAP server should be running on port 1636 with a single email in the inbox of the "inventor" user with the above message and a subject of your choosing

- Should be able to login to the IMAP server with the credentials
    - Username: `inventor@bashaway2k23.net`
    - Password: `123456`

**Starter code**

Starter `execute.sh`: (empty)

Extra npm deps used by the tests: `imap-simple@5.1.0`

**Test `test/index.test.js`**
```js
const imaps = require('imap-simple');
test('should validate if a mail is present in the inbox', async () => {
  let debugOutput = '';
  await new Promise((resolve) => {
    imaps.connect({
      imap: {
        user: "inventor@bashaway2k23.net",
        password: "123456",
        host: 'localhost',
        port: 1636,
        debug: (debug) => {
          debugOutput += debug;
        }
      }
    }).then(function (connection) {
      return connection.openBox('INBOX').then(function () {
        return connection.search(['UNSEEN'], {}).then(()=>{
          connection.end();
          resolve(debugOutput);
        });
      });
    });
  })
  expect(debugOutput).toContain("1 EXISTS");
  expect(debugOutput).toContain("[UNSEEN 1] Message 1 is the first unseen");
});
```

**Key idea:** `rnwood/smtp4dev` gives you SMTP and IMAP in one container (map IMAP 143 to 1636). Send the mail with nodemailer, or with `curl smtp://localhost:2525 --mail-from ... --mail-rcpt ... -T mail.txt`.

**Solution: team *SLIIT FOSS***
```bash
docker run -d --rm -it -p 3000:80 -p 2525:25 -p 1636:143 rnwood/smtp4dev:v3
sleep 10
node email.js
```
`email.js`:
```js
const nodemailer = require('nodemailer');

const transport = nodemailer.createTransport({
    host: "localhost",
    port: "2525",
});

new Promise((resolve, reject)=>{
    transport.sendMail({
        from: {
            name: 'Tech Baron',
            address: "tech.baron@bashaway2k23.net"
        },
        to: "inventor@bashaway2k23.net",
        text: "Technology knows no bounds.",
        subject: "Hello"
    }, (err)=>{
        if(err) reject(err)
        resolve()
    });
})
```

---

## 2022 challenges (questions only; the starter code was never published)

### autofill (2022 · easy)

**Autofill**

**Scenario**

Having to do the same tasks again and again is boring. It is same when it comes to filling same CLI prompts in Linux environments on daily basis. So it will be easier we can develop an answering script to provide answers for prompts.

**Task**

Develop a bash script to answer the following prompts created by questions.sh script:
```
Hi! Welcome to SLIIT Bashaway 2022.
Your team name:
No. of members in your team:
Your university:
```

You should develop a bash script to answer the above prompt as follows.
```
Hi! Welcome to SLIIT Bashaway 2022.
Your team name: <Your Team Name>
No. of members in your team: <Member count>
Your university: <University Name>

```

Then the provided bash script will create a file by your team name including your team details.
Eg: `team_name.txt`

You can test it by the following command:
`./questions.sh < ./script.sh`

### leaked-file (2022 · easy)

**Leaked file**

Chuti Ayya is doing a study session for his little brother and his friends. For this, he has prepared some resource files and has them stored in multiple ZIP files.

Chuti Ayya is very forgetful. So he stores all of his passwords in a file named password.txt .

Here's the kicker, Chuti Ayya realized he has accidently stored the password.txt in one of the ZIP files. He is asking for your help in resolving this situation.

**Task**
Copy every ZIP file except the ZIP file that includes the password.txt file; to the 'Output' directory.

**Input format**
In the Input directory, you will be given multiple ZIP files. One of which, includes the password.txt file.

**Output format**
The Output directory should have every ZIP file, except the ZIP file that includes password.txt

<br/>
<br/>

 # Docker build & run
```
 > docker build -t question1 .
 > docker run -it question1
 root@xxxxx: /usr/src/app# >  bash script.sh

 ```

### new-to-cli (2022 · easy)

**New to CLI**

ICT teacher of ABC school is working on familiarizing CLI tools among his students as most of the school students have no experience using CLI tools. As a start, he is thinking of grouping students in to multiple teams and let them experience the interactive nature of CLI through a simple Bash script. Each team will get access to a Linux PC with this script.

Your task is to develop that bash script for him. Teams are named as the team members preferred which may contain Uppercase, Lowercase, Numbers but not special characters execpt dash (-), underscore (_) and whitespace ( ).

This script should be interactive enough to prompt and get the following information from the team and save the data to a .txt file with team name. (eg: \<team_name>.txt. Whitespaces in middle of the group name should be replaced with underscore in the file name)
- Group Name
- Member Names
- Member NIC Numbers
- Member Phone Numbers
- Member Addresses

To get an idea about how long it took for each student to complete the prompted form, you should add timestamp at the following instances.
- Start of script
- End at entering Team Name.
- End of each students turn
- End of script

Make sure the team will not be needing to run the script more than once to enter the team details.

### whats-your-id (2022 · easy)

**What's your ID?**

Mrs Perera is a lecturer of ECD Univertsity. She teaches "English for Academic Purpose" to a class of First year students.

She has asked the students to submit a an assignment through the learning management system. The Students were supposed to name the file with their student ID but none of the students has done that.

Given that every student has included their student ID somewhere in the file, find the student ID and remane the file as \<STUDENT ID>.txt

**Input format**
The set of files will be in the "Files" directory.
Every file will include the Student ID in one line, the line will only have the student ID.
The student ID will be in the SIDXXXX format where X is a number between 0-9.

<br/>
<br/>

 # Docker build & run
```
 > docker build -t question1 .
 > docker run -it question1
 root@xxxxx: /usr/src/app# >  bash script.sh

 ```

### email-assistant (2022 · medium)

**The Email Assistant**

**Scenario**

You're a volunteer in a non-profit organization that conducts various humanitarian projects with celebrities. As a non-profit organization, the primary source of income will be sponsorships and partnerships from external individuals and organizations, including celebrities. Raising funds via sponsorships and partnerships is not easy. They have to reach many external entities, such as individuals and companies, within a short period by using only a few volunteers.

Their primary way of getting in touch with external entities is emailing. Whenever they initiate a project, they create a list of email addresses of external entities which may be interested in their project. Then they send a customized email for each by modifying an email template. It is very time-consuming as they have to create every email manually.

**Task**

As a volunteer in their organization with IT knowledge, they request you to do something to make it easier. So your task is to develop a bash script to modify the given email template (template.txt) and create customized email bodies refering the details given in the recipients.txt file. Content in the recipients.txt file takes the following format.

```
<Recipient's_Name>,<Recipient's_Email_Address>
<Personalized_Text_1>
<Personalized_Text_2>
<Personalized_Text_3>
<Blank_Line>
```

Information common to all the personalized emails tot be sent such as Sender's Name, Project Name, Project Description have to be entered as runtime arguments.

`./script.sh <Sender's_Name> <Project_Name> <Project_Description>`

Each personalized email body should be saved as a separate text file named by the recipient's email address as below.

`sales@company-name.com.txt`

Use the "emails" folder to store the personalized email text files.

### fix-the-indentation (2022 · medium)

**Fix the Indentation**

**Scenario**

Jessie is a newbie to Linux. However, he has learned some Bash scripting by following tutorials on the internet and trial and error. As we all know, Bash is not indentation sensitive. Jessie also ignores the indentations in his Bash scripts.

Suddenly, Jessie realized that his scripts were challenging to read because he had not followed the programming best practice of indenting. Now he is struggling to rearrange all his Bash scripts according to the correct indentations. To get this sorted out, Jessie seeks his teachers' help, who's Mr. White.

**Task**

Assume you are Mr. White. Your task is to develop a Python script to fix all the selection (if/else) and repetition (loops) related to indentation issues in Jessie's Bash scripts. Also, use a single blank line above and below every loop so that Jessie can refer to his codes easier, and he can continue his self-learning.

Jessie's original scripts are stored in a folder by his name. You should save indentation fixed bash scripts into a folder by the name "White." Newly created files should have the same name as their original counterparts.

### it-admin (2022 · medium)

**IT Admin**

**Scenario**

An educational institute maintains a Linux server to let its students practice basic Linux skills. As the IT admin of the institute, create the given list of user accounts with assigned usernames and add them to their respective user group. Each student should get their relevant learning materials in their user account home directory once the account is ready to use.

Usernames and their respective group name is given in UserList.txt as follows.
```
user1,groupA
user2,groupA
user3,groupB
```

Learning materials for each student group is in the following folders.
- Learning_Materials_A
- Learning_Materials_B

**Task**

Your task is to create a Bash script to automate this boring process as follows.

For each user account, you have to autogenerate a password with at least 8 characters. These autogenerated passwords should have minimum 2 of the following 4 different character types
- uppercase
- lowercase
- numbers

Once the users are created, UserList.txt file should be updated with the newly generated passwords as follows.
```
user1,groupA,password1
user2,groupA,password2
user3,groupB,password3
```

### url-scraper (2022 · medium)

**URL Scraper**

**Scenario**

You and Nimal are freelance web developers who usually deliver low-budget projects. To minimize the budget, you always try using non-copyrighted media files (images, videos etc.) and minimum hosting space. Therefore, you have been using media files hosted on the internet instead of hosting the media files among the website artifacts. These media files are 3rd-party hosted public image sets, etc., without any CORS issues.

After some time, your clients complain about missing images on their websites. After checking up on their complaints, you two realize that some of the 3rd-party hosted images have been removed or deleted. To avoid such incidents further, you two plan to download all the media files you have used in your projects and host them in a CDN.

You two understand this as a tedious task, so you suggest Nimal automate the whole thing. As a skilled Linux user, you agree to develop an automation script to list URLs of the media files included in each project. At the same time, Nimal takes care of automating downloading those media files using the URLs you will provide.

**Task**

Develop a Bash script to list the URLs of all the media files used in a given website project directory. Assume that all these websites are developed only using HTML and CSS where no JavaScript or other programming language is used. The script should append the URLs to a text file saved as "url_list.txt."

**Hint**

As the media files are not hosted among the website artifacts but some remote source, the best way to figure a way out is to follow the source.

**Instructions**

Bash script file and the url_list.txt files are already copied into the "bashaway-test-site" project folder for your ease.

### certificate-generator (2022 · hard)

**Certificate Generator**

**Scenario**

Moordale High School is organizing its annual prize-giving ceremony. The organizing committee has planned to distribute certificates to more than 400 students resulting in 400 certificates awarded. So manually completing all these certificates is a tedious task. So school management decided to get help from Otis Milburn, who is good at automation, to get this automated.

**Task**

Assume you are Otis and develop a script to generate the certificates using the following assets automatically.
1. Certificate Template (certificate.png)
2. Signature (signature.png)
3. Student Name (candidates.csv)
4. Font (basic-font.ttf)

Font should be in Black: RGB(0,0,0) in size 16. For each student in the name list, a separate PDF file should be a seperate PDF file generated in the output folder.

### event-rush (2022 · hard)

**Event Rush**

**Scenario**

Clay Jensen and Hannah Baker are organizing a suicide prevention awareness event at Liberty High School, Evergreen County, California. As a part of the event, they are planning to make photo badges which can be sticked on shoulder as a fancy item. To make the photo badges, they will be pasting a rounded sticker on a rounded plastic plate with a clip. They have already bought the plastic clip but they have only a few hours left to get done the stickers.

Stickers should be rounded printed on A4 size sticker papers. Each A4 sheet can hold up to 8 stickers as 4 rows with 2 columns in each in portrait orientation. Stickers should contain the images provided in the "images" folder. Once the 8 images are placed correctly in the layout, the finalized layout should be exported as a single PDF file.

To get the job run in the run, they invite Tyler who is familair with tech.

**Task**

Assume you are Tyler and develop a Python script to generate the PDF with all the provided images in the images directory. The PDF should contain all the given square shaped images in rounded shape at 8 stickers per 1 sheet. The 8 stickers should be placed on a portrait-oriented sheet as 4 rows with 2 columns.

For further understanding of the layout, refer to the following image.

[image omitted]

### fix-the-indentation (2022 · hard)

**Fix the Indentation**

**Scenario**

Jessie is a newbie to Linux. However, he has learned some Bash scripting by following tutorials on the internet and trial and error. As we all know, Bash is not indentation sensitive. Jessie also ignores the indentations in his Bash scripts.

Suddenly, Jessie realized that his scripts were challenging to read because he had not followed the programming best practice of indenting. Now he is struggling to rearrange all his Bash scripts according to the correct indentations. To get this sorted out, Jessie seeks his teachers' help, who's Mr. White.

**Task**

Assume you are Mr. White. Your task is to develop a Bash script to fix all the selection (if/else) and repetition (loops) related to indentation issues in Jessie's Bash scripts. Also, use a single blank line above and below every loop so that Jessie can refer to his codes easier, and he can continue his self-learning.

Jessie's original scripts are stored in a folder by his name. You should save indentation fixed bash scripts into a folder by the name "White." Newly created files should have the same name as their original counterparts.

### place-the-footer (2022 · hard)

**Place the Footer**

**Scenario**

Photographers always add a footer to their photos to add the event details and show their credits in photography. Nowadays, photographers capture hundreds of photos at every event, making their life harder when adding footers to their photos.

**Task**

Develop a script to attach the provided footer (footer.png) to all of the images in the "input" folder. Make sure to save your output images into the "output" folder. When adding the footer, make sure about the following criteria.
1. footer should be fit into image
2. size of the given images are not fixed.
3. footer should be transparent.

### mini-studio (2022 · extreme)

**Mini-Studio**

**Scenario**

Ryder is a newbie YouTuber who runs a mini studio in Groove Street, Los Santos. He creates lyrics videos for the latest music tracks and publishes them on YouTube. He also monetizes them to earn money from YouTube. These videos are pretty simple as the lyrics are displayed in real-time in front of a static image.

 In the early days, Ryder used Windows Movie Maker to create the lyrics videos. Still, it takes him a few hours to correctly match the lyrics and the music to create the music video. Therefore he invites his neighbor Carl Johnson (CJ). He is a multi-talented guy, and this mission is about finding a way to make Ryder's job easier so he can make more money quickly.

**Task**

Assume you are CJ, skilled in Python automation. To pass this mission, you must develop a python script to automate the lyric video generation process by providing the background image, music track (mp3 file), lyrics and timeline. The script should generate a lyric video with that provided data where the output should be in mp4 format.

Refer to the given dataset for further understanding.
