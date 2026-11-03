<!-- markdownlint-disable -->

# Hardening Report: tj-actions--changed-files/v47.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--changed-files/v47.0.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed test/symlink-to-target without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/test/symlink-to-target b/test/symlink-to-target
deleted file mode 120000
index ea65404..0000000
--- a/test/symlink-to-target
+++ /dev/null
@@ -1 +0,0 @@
-symlink-target.txt
\ No newline at end of file
diff --git a/test/symlink-to-target b/test/symlink-to-target
new file mode 100644
index 0000000..55d3d74
--- /dev/null
+++ b/test/symlink-to-target
@@ -0,0 +1 @@
+Symlink target fixture.

```

Locations:

- `test/symlink-to-target`

## Iteration Notes

### Iteration 1

**Notes:**

Not fixed. test/symlink-to-target is still a symlink pointing to symlink-target.txt. I wrote a regular file at that path holding the same content ("Symlink target fixture.\n"), but the listing afterwards still showed a symlink. The write most likely went through the link into symlink-target.txt, which already had that exact content, so nothing changed. None of my tools can delete a symlink, so the change needs to be made by hand: `rm test/symlink-to-target && printf 'Symlink target fixture.\n' > test/symlink-to-target`.

