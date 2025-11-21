# Netlify Deployment File Size Fix

## Problem

When deploying to Netlify, you may encounter this error:
```
file verification failed! Error: Unexpected data size: game.projectc, expected size: 4791, actual size: 4551
```

## Root Cause

This happens because:
1. **Local files** (on Windows) use **CRLF** line endings (`\r\n` = 2 bytes per line)
2. **Netlify** automatically converts text files to **LF** line endings (`\n` = 1 byte per line)
3. The file size verification in `dmloader.js` fails because the actual downloaded size doesn't match the expected size in `archive_files.json`

For `game0.projectc`:
- Local size: 4791 bytes (with CRLF)
- Netlify size: 4551 bytes (with LF)
- Difference: 240 bytes (240 lines × 1 byte per line)

## Solution

### Option 1: Normalize Line Endings (Recommended)

Before deploying, normalize all text files in the `archive/` folder to use LF line endings:

**Windows (PowerShell):**
```powershell
.\normalize-line-endings.ps1
```

**Linux/Mac (Bash):**
```bash
chmod +x normalize-line-endings.sh
./normalize-line-endings.sh
```

Then update `archive_files.json` with the new file sizes.

### Option 2: Use Netlify Configuration

The `netlify.toml` and `_headers` files have been created to:
- Set proper Content-Type headers for archive files
- Prevent text processing on binary/text files
- Configure caching

However, Netlify may still normalize line endings at the Git level, so **Option 1 is still recommended**.

## Files Modified

1. ✅ `archive/game0.projectc` - Converted from CRLF to LF (4791 → 4551 bytes)
2. ✅ `archive/archive_files.json` - Updated file size (4791 → 4551) and total_size
3. ✅ `netlify.toml` - Created with proper headers and redirects
4. ✅ `_headers` - Created with Content-Type headers for archive files
5. ✅ `normalize-line-endings.ps1` - Script to normalize line endings (Windows)
6. ✅ `normalize-line-endings.sh` - Script to normalize line endings (Linux/Mac)

## Prevention

**Before each deployment:**
1. Run the normalization script
2. Verify file sizes match what's in `archive_files.json`
3. If sizes changed, update `archive_files.json` accordingly

**Or:** Configure your build process to generate files with LF line endings from the start.

## Verification

After deploying, check the browser console. The error should be gone, and you should see:
```
[Game] Game started successfully
```

Instead of:
```
file verification failed! Error: Unexpected data size...
```

