# Extraction tooling — code snippets

**All code below is authored within this research project** (phases 1–3) and is
provided for reproducibility. These scripts only **read** the installed binaries and
write to the research workspace — they never modify the originals, and none of them
touch exam-security mechanisms. Run them from a trusted directory against absolute
target paths.

## 1. Full embedded-asset scan (brotli streams in a Tauri binary)

Tauri release builds embed frontend assets via `tauri-codegen` "embeds, hashes, and
compresses" (official docs, Context7). Brotli streams carry **no magic signature**, so
the scan brute-forces candidate offsets and keeps every offset that decompresses
cleanly. `zlib.brotliDecompressSync` fails fast on garbage (~30 000 offsets ≈ 16 ms;
the full 44 MB binary takes a couple of minutes).

```js
// extract_assets.js — full scan; see source/extraction/EXTRACTION-REPORT.md for results
const fs = require('fs'), zlib = require('zlib'), crypto = require('crypto');
const exe = fs.readFileSync('D:/Digiexam/digiexam.exe');
const sha = (b) => crypto.createHash('sha256').update(b).digest('hex').toUpperCase();

const hits = [];
for (let off = 0; off < exe.length - 4; off++) {
  try {
    const d = zlib.brotliDecompressSync(exe.subarray(off));
    if (d.length >= 24) hits.push({ off, size: d.length, data: d });
  } catch (e) { /* garbage, expected */ }
}
// sniff + save
for (const h of hits) {
  const head = h.data.subarray(0, 300).toString('latin1');
  const kind =
    head.startsWith('wOF2') ? 'font/woff2' :
    head.startsWith('\x89PNG') ? 'image/png' :
    /^<!DOCTYPE html/i.test(head) ? 'text/html' :
    /^[[{]/.test(head) ? 'application/json' : 'application/octet-stream';
  console.log('0x' + h.off.toString(16), h.size, kind, sha(h.data));
}
```

## 2. Asset-path enumeration (locate the embedded asset table)

```js
const exe = fs.readFileSync('D:/Digiexam/digiexam.exe').toString('latin1');
const re = /[\/A-Za-z0-9_.\-]{4,120}\.(?:woff2?|png|svg|ico|json|html|css|js|wasm)/g;
const seen = new Map();
let m; while ((m = re.exec(exe)) !== null) seen.set(m[0], (seen.get(m[0]) || 0) + 1);
console.log([...seen.keys()].sort());
```

## 3. Verify an extracted asset against the signed binary (phase-2 provenance chain)

```js
// Re-extract at a recorded offset and hash-compare with the preserved copy.
const d = zlib.brotliDecompressSync(exe.subarray(30807145));   // index.html
console.log(sha(d)); // 5C2300BCD2FA739F4A530884786A27144C4A111FF2B33C5043EA63280E01EE76
```

## 4. Binary marker scan (platform identification, phase 2)

Direct `Regex.Matches` over full-file ASCII + UTF-16LE decodes:

```powershell
$b = [System.IO.File]::ReadAllBytes('D:\Digiexam\digiexam.exe')
$ascii = [System.Text.Encoding]::ASCII.GetString($b)
$uni   = [System.Text.Encoding]::Unicode.GetString($b)
'tauri','WebView2','Electron','BSJB' | ForEach-Object {
  "$_ : " + ([regex]::Matches($ascii, $_).Count + [regex]::Matches($uni, $_).Count)
}
```

**Pitfall (recorded in doc 05):** wrapping the count in a ScriptBlock and calling
`.Invoke(...).Count` returns the *wrapper's* count (1), not the match count — use
inline `Regex.Matches`.

## 5. PE header + CLR check (is it .NET? is it x64?)

```powershell
$fs = [System.IO.FileStream]::new($path, 'Open', 'Read')
$br = [System.IO.BinaryReader]::new($fs)
$fs.Seek(0x3C, 'Begin') | Out-Null; $peOff = $br.ReadInt32()      # e_lfanew
$fs.Seek($peOff + 4, 'Begin') | Out-Null; '{0:X4}' -f $br.ReadUInt16()  # machine (0x8664 = x64)
# CLR check: count occurrences of 'BSJB' (CLR metadata magic) in the raw bytes — 0 = not .NET
```

## 6. Live service state (Windows SCM, read-only)

```powershell
Get-CimInstance Win32_Service | Where-Object Name -like '*digiexam*' |
  Format-List Name, PathName, StartMode, State
# Result: DigiexamWindowsService · D:\Digiexam\service.exe · Auto · Running
```

## 7. Offset-pinning helper (verify a string literal at a raw file offset)

```js
const latin = fs.readFileSync('D:/Digiexam/digiexam.exe').toString('latin1');
const at = (off, len) => latin.substring(off, off + len);   // NOTE: string → substring, not subarray
console.log(at(28200792, 28) === 'digiexam_toolbox_service_ipc'); // true
```

## 8. Lockdown component locator (raw offsets for embedded module paths)

Enumerates every occurrence of a literal (Rust panic-location metadata repeats per
call site) and walks prefix regexes to list complete module paths — produces the raw
offsets published in [`../analysis/phase-3/14-lockdown-raw-evidence.md`](../analysis/phase-3/14-lockdown-raw-evidence.md):

```js
const fs = require('fs');
const buf = fs.readFileSync('D:/Digiexam/service.exe').toString('latin1');
const findAll = (s, needle) => { const o = []; let i = s.indexOf(needle);
  while (i !== -1) { o.push(i); i = s.indexOf(needle, i + 1); } return o; };
// exact literals:
for (const lit of ['lockdown-core\\src\\handler\\mod.rs', 'unauthorized IPC client attempted to connect'])
  console.log(lit, '->', findAll(buf, lit).map(n => '0x' + n.toString(16)));
// prefix enumeration (complete module paths):
const re = /lockdown-core\\src\\handlers\\windows\\[a-z0-9_\\]+\.rs/g; let m;
while ((m = re.exec(buf)) !== null) console.log('0x' + m.index.toString(16), m[0]);
```

Raw byte context (hex + ASCII) at a located offset — note `substring` on the Latin-1
*string*, and that both the label **and** the data must use the same absolute offset:

```js
function dump(s, off, len) {
  let out = '';
  for (let i = 0; i < len; i += 16) {
    const chunk = s.substring(off + i, off + Math.min(i + 16, len));  // absolute offset!
    let hx = '', as = '';
    for (const ch of chunk) { const c = ch.charCodeAt(0);
      hx += c.toString(16).padStart(2, '0') + ' ';
      as += (c >= 32 && c < 127) ? ch : '.'; }
    out += (off + i).toString(16).padStart(8, '0') + '  ' + hx.padEnd(48) + ' |' + as + '|\n';
  }
  return out;
}
```

## Not included (and why)

- **No reconstructed lockdown/enforcement code.** The embedded frontend is minified
  build output (not original source) and is preserved locally under the policy in
  [`recovered-source/.gitignore`](../recovered-source/.gitignore); native lockdown
  logic exists only as compiled Rust in signed binaries. Reproducing enforcement
  internals is outside this repository's scope (see
  [`../documentation/legal-and-scope.md`](../documentation/legal-and-scope.md)).
- The beautified (reconstructed) copy of the frontend bundle is generated locally by
  `npx js-beautify` but not published, for the same copyright reason.
