# Security Checklist - nanobot

Quick reference checklist for deploying nanobot securely.

## 🔒 Pre-Deployment Security Checklist

### Essential (Must Do Before Production)

- [ ] **Configure Access Control**
  - Set `allowFrom` lists for ALL enabled channels (Telegram, Discord, WhatsApp, Email, etc.)
  - Empty `allowFrom` = ALLOW EVERYONE (insecure!)
  - Get your user IDs from respective platforms

- [ ] **Enable Workspace Restrictions**
  - Set `"restrictToWorkspace": true` in config under `tools` section
  - Prevents file access outside designated workspace
  - Protects sensitive system files

- [ ] **Secure Configuration File**
  - Ensure `~/.nanobot/config.json` has permissions `0600` (automatically set in latest version)
  - Never commit config file to version control
  - Keep API keys secret

- [ ] **Use Strong API Keys**
  - Generate separate API keys for production vs development
  - Use keys with spending limits if available
  - Rotate keys regularly (every 90 days minimum)

- [ ] **Review Shell Tool Configuration**
  - Understand that `exec` tool can run shell commands
  - Consider disabling if not needed
  - Review dangerous command patterns in code

### Recommended (Should Do)

- [ ] **Run Latest Version**
  - Update to version with security fixes (2026-02-09 or later)
  - Check for security advisories regularly
  - Subscribe to GitHub notifications

- [ ] **Use Docker (Recommended)**
  - Run in container for isolation
  - Uses non-root user (in latest version)
  - Easier to update and rollback

- [ ] **Monitor Logs**
  - Check `~/.nanobot/logs/` regularly
  - Watch for security warnings (⚠️ symbols)
  - Look for unauthorized access attempts
  - Set up log rotation

- [ ] **Set Resource Limits**
  - Configure `exec.timeout` (default 60s, consider 30s for production)
  - Monitor API usage and costs
  - Set spending alerts with your LLM provider

- [ ] **Use HTTPS Only**
  - Verify all API endpoints use HTTPS
  - Check log warnings for HTTP connections
  - Configure proxy if needed for China/restricted regions

### Best Practices (Nice to Have)

- [ ] **Audit Regular Access**
  - Review who has access to bot
  - Remove unused user IDs from `allowFrom` lists
  - Document access changes

- [ ] **Backup Configuration**
  - Keep encrypted backup of config file
  - Store separately from production system
  - Document recovery process

- [ ] **Network Isolation**
  - Run on isolated network if possible
  - Use firewall rules to restrict outbound connections
  - Consider VPN for sensitive deployments

- [ ] **Dependencies**
  - Run `pip install --upgrade nanobot-ai` regularly
  - Check `npm audit` for bridge dependencies
  - Enable GitHub Dependabot alerts

- [ ] **Incident Response Plan**
  - Know how to revoke API keys quickly
  - Have rollback plan ready
  - Document security contacts

---

## 🚨 Security Warnings to Watch For

When you see these in logs, take action:

### ⚠️ Critical Warnings

```
⚠️  SECURITY WARNING: Channel 'telegram' has no access control configured.
Anyone can interact with this bot!
```
**Action**: Add `allowFrom` list to config immediately

```
Error: Command blocked by safety guard (dangerous pattern detected)
```
**Action**: Review what command was attempted and by whom

```
Error: Path outside allowed directory
```
**Action**: Check if legitimate or attack attempt

### ⚠️ Important Warnings

```
⚠️  Using HTTP (unencrypted) connection to http://example.com
```
**Action**: Verify if intentional, prefer HTTPS

```
Access denied for sender 123456 on channel telegram
```
**Action**: Verify this is expected, investigate if not

```
Warning: Could not set restrictive permissions on config
```
**Action**: Manually set permissions: `chmod 600 ~/.nanobot/config.json`

---

## 📋 Production Configuration Template

Save as `~/.nanobot/config.json`:

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.nanobot/workspace",
      "model": "anthropic/claude-opus-4-5",
      "max_tokens": 8192,
      "temperature": 0.7,
      "max_tool_iterations": 20
    }
  },
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-v1-YOUR_KEY_HERE"
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "allowFrom": ["123456789"]
    }
  },
  "tools": {
    "restrictToWorkspace": true,
    "exec": {
      "timeout": 30
    }
  },
  "gateway": {
    "host": "127.0.0.1",
    "port": 18790
  }
}
```

**Important Notes**:
- Replace `YOUR_KEY_HERE` and `YOUR_BOT_TOKEN` with actual credentials
- Add your Telegram user ID to `allowFrom` (get from @userinfobot)
- `host: "127.0.0.1"` prevents external access (use "0.0.0.0" only if needed)
- After saving, run: `chmod 600 ~/.nanobot/config.json`

---

## 🐳 Docker Deployment Checklist

- [ ] **Use Latest Image**
  ```bash
  docker build -t nanobot .
  ```

- [ ] **Mount Config Securely**
  ```bash
  docker run -v ~/.nanobot:/home/nanobot/.nanobot nanobot
  ```

- [ ] **Verify Non-Root User**
  ```bash
  docker run nanobot whoami
  # Should output: nanobot (not root)
  ```

- [ ] **Restrict Network**
  ```bash
  docker run --network=isolated nanobot
  ```

- [ ] **Set Resource Limits**
  ```bash
  docker run --memory=1g --cpus=1 nanobot
  ```

---

## 🔍 Quick Security Audit Commands

Run these periodically:

```bash
# Check config file permissions
ls -l ~/.nanobot/config.json
# Should show: -rw------- (600)

# Review recent logs for warnings
grep "WARNING" ~/.nanobot/logs/*.log | tail -20

# Check who has access (shows all allowFrom lists)
cat ~/.nanobot/config.json | grep -A 5 "allowFrom"

# Verify no API keys in git
git grep -i "api.key\|token.*:" 2>/dev/null
# Should return nothing

# Check Python dependencies for known vulnerabilities
pip install pip-audit
pip-audit

# Check Node.js dependencies (for WhatsApp bridge)
cd ~/.nanobot/bridge && npm audit
```

---

## ⚡ Quick Fixes

### Fix: Config file has wrong permissions
```bash
chmod 600 ~/.nanobot/config.json
chmod 700 ~/.nanobot
```

### Fix: Anyone can access my bot
Edit `~/.nanobot/config.json`:
```json
{
  "channels": {
    "telegram": {
      "allowFrom": ["YOUR_USER_ID_HERE"]
    }
  }
}
```

### Fix: Bot can access files anywhere
Edit `~/.nanobot/config.json`:
```json
{
  "tools": {
    "restrictToWorkspace": true
  }
}
```

### Fix: Suspicious activity detected
```bash
# 1. Stop the bot immediately
pkill -f nanobot

# 2. Revoke API keys (go to provider websites)

# 3. Review logs
grep -i "error\|warning\|denied" ~/.nanobot/logs/*.log

# 4. Reset config with new keys
vim ~/.nanobot/config.json

# 5. Restart with logging
nanobot gateway --logs
```

---

## 📞 Security Resources

- **Audit Report**: `/SECURITY_AUDIT_REPORT.md`
- **Improvements**: `/SECURITY_IMPROVEMENTS.md`
- **Security Policy**: `/SECURITY.md`
- **Issues**: https://github.com/HKUDS/nanobot/issues
- **Security Advisory**: https://github.com/HKUDS/nanobot/security/advisories

---

## ✅ Validation

After configuration, verify:

```bash
# Start bot in test mode
nanobot status

# Send test message (should work)
nanobot agent -m "Hello"

# Check logs for warnings
tail -f ~/.nanobot/logs/*.log

# Verify access control (should deny unknown users)
# Try to access from unauthorized account
```

---

**Last Updated**: 2026-02-09  
**Version**: Applies to nanobot v0.1.3.post5 and later with security fixes
