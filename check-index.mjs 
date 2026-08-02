#!/usr/bin/env node
/*
  Jobbin pre-deploy check.
  Runs on every push. Fails the build if index.html looks broken,
  so Cloudflare never publishes a file that won't load.
*/

import { readFileSync, writeFileSync, mkdtempSync } from "node:fs";
import { execFileSync } from "node:child_process";
import { tmpdir } from "node:os";
import { join } from "node:path";

const FILE = "index.html";
const fail = [];
const pass = [];

let html;
try {
  html = readFileSync(FILE, "utf8");
} catch {
  console.error(`Could not read ${FILE}. Is it still at the root of the repo?`);
  process.exit(1);
}

/* 1 - Size. A bad paste usually truncates the file. */
const MIN_KB = 100;
const kb = Math.round(html.length / 1024);
if (kb < MIN_KB) {
  fail.push(`${FILE} is only ${kb} KB, expected at least ${MIN_KB} KB. Looks truncated.`);
} else {
  pass.push(`Size looks right (${kb} KB).`);
}

/* 2 - Tag balance. Divs are counted in the markup only, ignoring script contents. */
const markup = html.replace(/<script[\s\S]*?<\/script\s*>/gi, "");
const count = (text, re) => (text.match(re) || []).length;
const balance = [
  ["script", html, /<script\b/gi, /<\/script\s*>/gi],
  ["style", html, /<style\b/gi, /<\/style\s*>/gi],
  ["div", markup, /<div\b/gi, /<\/div\s*>/gi],
];
for (const [tag, text, open, close] of balance) {
  const o = count(text, open);
  const c = count(text, close);
  if (o !== c) fail.push(`Unbalanced <${tag}> tags: ${o} opened, ${c} closed.`);
  else pass.push(`<${tag}> tags balanced (${o}).`);
}

/* 3 - The JavaScript actually parses. This is the one that catches a bad paste. */
const scripts = [...html.matchAll(/<script\b([^>]*)>([\s\S]*?)<\/script\s*>/gi)]
  .filter((m) => !/\bsrc\s*=/i.test(m[1]))
  .map((m) => m[2])
  .filter((code) => code.trim().length);

const dir = mkdtempSync(join(tmpdir(), "jobbin-"));
scripts.forEach((code, i) => {
  const path = join(dir, `inline-${i}.js`);
  writeFileSync(path, code);
  try {
    execFileSync(process.execPath, ["--check", path], { stdio: "pipe" });
  } catch (err) {
    const msg = String(err.stderr || "").split("\n").slice(0, 6).join("\n");
    fail.push(`Inline script #${i + 1} has a syntax error:\n${msg}`);
  }
});
if (scripts.length) pass.push(`Checked ${scripts.length} inline script block(s).`);

/* 4 - Things that must never go missing. */
const required = [
  ["unpkg.com/lucide", "the Lucide icons script"],
  ["accounts.google.com/gsi/client", "the Google sign-in script (Drive backup needs it)"],
  ["jobbit-favicon.svg", "the favicon link"],
  ["ebon-Heir", "the credit line"],
];
for (const [needle, label] of required) {
  if (!html.includes(needle)) fail.push(`Missing ${label} ("${needle}").`);
  else pass.push(`Found ${label}.`);
}

/* ---- report ---- */
console.log("Passed:");
for (const p of pass) console.log("  OK   " + p);
if (fail.length) {
  console.log("\nProblems:");
  for (const f of fail) console.log("  FAIL " + f);
  console.log(`\n${fail.length} problem(s) found. Not safe to deploy.`);
  process.exit(1);
}
console.log("\nAll checks passed.");
