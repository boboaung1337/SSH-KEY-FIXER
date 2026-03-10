<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SSH Key Fixer - One Click Fix</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: #0d1117;
            font-family: 'Courier New', monospace;
            padding: 20px;
        }

        .container {
            max-width: 1000px;
            margin: 0 auto;
        }

        h1 {
            color: #2f81f7;
            font-size: 24px;
            margin-bottom: 20px;
            border-left: 4px solid #2f81f7;
            padding-left: 15px;
        }

        .editor {
            background: #161b22;
            border: 1px solid #30363d;
            border-radius: 6px;
            margin-bottom: 20px;
        }

        .editor-header {
            background: #21262d;
            padding: 10px 15px;
            border-bottom: 1px solid #30363d;
            color: #8b949e;
            font-size: 14px;
        }

        textarea {
            width: 100%;
            height: 300px;
            padding: 15px;
            background: #0d1117;
            border: none;
            color: #7ee787;
            font-family: 'Courier New', monospace;
            font-size: 13px;
            line-height: 1.6;
            resize: vertical;
        }

        textarea:focus {
            outline: none;
        }

        textarea[readonly] {
            color: #79c0ff;
        }

        .info-bar {
            background: #161b22;
            border: 1px solid #30363d;
            border-radius: 6px;
            padding: 15px;
            margin: 20px 0;
            display: flex;
            justify-content: space-between;
            color: #8b949e;
        }

        .button {
            background: #238636;
            color: white;
            border: none;
            padding: 15px;
            font-size: 16px;
            font-weight: bold;
            border-radius: 6px;
            cursor: pointer;
            width: 100%;
            margin-bottom: 20px;
            font-family: 'Courier New', monospace;
        }

        .button:hover {
            background: #2ea043;
        }

        .button:active {
            transform: scale(0.99);
        }

        .copy-btn {
            background: #21262d;
            color: #2f81f7;
            border: 1px solid #30363d;
            padding: 10px;
            border-radius: 6px;
            cursor: pointer;
            font-family: 'Courier New', monospace;
            margin-top: 10px;
        }

        .copy-btn:hover {
            background: #30363d;
        }

        .footer {
            text-align: center;
            color: #8b949e;
            margin-top: 30px;
            font-size: 12px;
        }

        .badge {
            background: #1f6feb;
            color: white;
            padding: 2px 8px;
            border-radius: 12px;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔑 SSH KEY FIXER</h1>
        
        <div class="editor">
            <div class="editor-header">
                📥 PASTE YOUR BROKEN KEY HERE
            </div>
            <textarea id="input" placeholder="Paste your SSH key with \n and \/ problems..."></textarea>
        </div>

        <div class="info-bar">
            <span>🔍 Status: <span id="status">Waiting for input...</span></span>
            <span>📊 Issues: <span id="issueCount">0</span></span>
        </div>

        <button class="button" onclick="fixKey()">
            🔧 FIX KEY NOW
        </button>

        <div class="editor">
            <div class="editor-header">
                📤 FIXED KEY (ready to use)
            </div>
            <textarea id="output" readonly placeholder="Fixed key will appear here..."></textarea>
        </div>

        <button class="copy-btn" onclick="copyKey()" style="width: 100%;">
            📋 COPY TO CLIPBOARD
        </button>

        <div class="footer">
            ⚡ Auto-fixes: \n • \r • \t • \/ • \\ and all escape sequences
        </div>
    </div>

    <script>
        const input = document.getElementById('input');
        const output = document.getElementById('output');
        const status = document.getElementById('status');
        const issueCount = document.getElementById('issueCount');

        // Update status on input
        input.addEventListener('input', updateStatus);

        function updateStatus() {
            const text = input.value;
            const issues = countIssues(text);
            
            if (text.trim() === '') {
                status.textContent = 'Empty';
                status.style.color = '#8b949e';
            } else if (issues === 0) {
                status.textContent = 'Clean';
                status.style.color = '#7ee787';
            } else {
                status.textContent = `${issues} issues detected`;
                status.style.color = '#f85149';
            }
            
            issueCount.textContent = issues;
        }

        function countIssues(text) {
            let count = 0;
            count += (text.match(/\\n/g) || []).length;
            count += (text.match(/\\r/g) || []).length;
            count += (text.match(/\\t/g) || []).length;
            count += (text.match(/\\\//g) || []).length;  // Count \/
            count += (text.match(/\\\\/g) || []).length;
            return count;
        }

        function fixKey() {
            let text = input.value;
            
            if (!text.trim()) {
                alert('Paste a key first!');
                return;
            }

            // CRITICAL FIX: Process the string character by character
            // This handles ALL escape sequences properly
            let result = '';
            let i = 0;
            
            while (i < text.length) {
                if (text[i] === '\\' && i + 1 < text.length) {
                    // Handle escape sequences
                    switch (text[i + 1]) {
                        case 'n':
                            result += '\n';
                            i += 2;
                            break;
                        case 'r':
                            result += '\r';
                            i += 2;
                            break;
                        case 't':
                            result += '\t';
                            i += 2;
                            break;
                        case '/':
                            result += '/';  // This fixes \/ to /
                            i += 2;
                            break;
                        case '\\':
                            result += '\\';  // This fixes \\ to \
                            i += 2;
                            break;
                        default:
                            result += text[i];
                            i++;
                    }
                } else {
                    result += text[i];
                    i++;
                }
            }

            // Now clean up the result
            let lines = result.split('\n');
            let cleanedLines = [];
            let inHeader = false;
            let header = '';
            let footer = '';
            let content = [];

            for (let line of lines) {
                // Remove carriage returns
                line = line.replace(/\r$/, '');
                
                // Skip empty lines
                if (line.trim() === '') continue;
                
                // Detect header
                if (line.includes('BEGIN') && line.includes('KEY')) {
                    header = '-----BEGIN RSA PRIVATE KEY-----';
                    inHeader = true;
                }
                // Detect footer
                else if (line.includes('END') && line.includes('KEY')) {
                    footer = '-----END RSA PRIVATE KEY-----';
                    inHeader = false;
                }
                // Content lines
                else if (inHeader) {
                    // Remove any remaining backslashes or special chars
                    let clean = line.replace(/[^A-Za-z0-9\+\/=]/g, '');
                    if (clean.length > 0) {
                        content.push(clean);
                    }
                }
            }

            // Rebuild the key with proper formatting
            if (header && footer && content.length > 0) {
                let base64 = content.join('');
                
                // Fix padding
                while (base64.length % 4 !== 0) {
                    base64 += '=';
                }

                // Format to 64 chars per line
                let wrapped = [];
                for (let i = 0; i < base64.length; i += 64) {
                    wrapped.push(base64.substr(i, 64));
                }

                // Build final key
                let final = header + '\n' + wrapped.join('\n') + '\n' + footer;
                output.value = final;
                
                status.textContent = 'Fixed!';
                status.style.color = '#7ee787';
            } else {
                // If structure not found, just return cleaned text
                output.value = result;
            }

            updateStatus();
        }

        function copyKey() {
            if (!output.value) {
                alert('No key to copy!');
                return;
            }
            
            output.select();
            document.execCommand('copy');
            
            // Visual feedback
            const btn = document.querySelector('.copy-btn');
            btn.textContent = '✅ COPIED!';
            btn.style.background = '#238636';
            btn.style.color = 'white';
            
            setTimeout(() => {
                btn.textContent = '📋 COPY TO CLIPBOARD';
                btn.style.background = '#21262d';
                btn.style.color = '#2f81f7';
            }, 2000);
        }

        // Load sample on page load
        window.onload = function() {
            input.value = '-----BEGIN RSA PRIVATE KEY-----\\nMIIEogIBAAKCAQEAs7sytDE++NHaWB9e+NN3V5t1DP1TYHc+4o8D362l5Nwf6Cpl\\nmR4JH6n4Nccdm1ZU+qB77li8ZOvymBtIEY4Fm07X4Pqt4zeNBfqKWkOcyV1TLW6f\\n87s0FZBhYAizGrNNeLLhB1IZIjpDVJUbSXG6s2cxAle14cj+pnEiRTsyMiq1nJCS\\ndGCc\\/gNpW\\/AANIN4vW9KslLqiAEDJfchY55sCJ5162Y9+I1xzqF8e9b12wVXirvN\\no8PLGnFJVw6SHhmPJsue9vjAIeH+n+5Xkbc8\\/6pceowqs9ujRkNzH9T1lJq4Fx1V\\nvi93Daq3bZ3dhIIWaWafmqzg+jSThSWOIwR73wIDAQABAoIBADHwl\\/wdmuPEW6kU\\nvmzhRU3gcjuzwBET0TNejbL\\/KxNWXr9B2I0dHWfg8Ijw1Lcu29nv8b+ehGp+bR\\/6\\npKHMFp66350xylNSQishHIRMOSpydgQvst4kbCp5vbTTdgC7RZF+EqzYEQfDrKW5\\n8KUNptTmnWWLPYyJLsjMsrsN4bqyT3vrkTykJ9iGU2RrKGxrndCAC9exgruevj3q\\n1h+7o8kGEpmKnEOgUgEJrN69hxYHfbeJ0Wlll8Wort9yummox\\/05qoOBL4kQxUM7\\nVxI2Ywu46+QTzTMeOKJoyLCGLyxDkg5ONdfDPBW3w8O6UlVfkv467M3ZB5ye8GeS\\ndVa3yLECgYEA7jk51MvUGSIFF6GkXsNb\\/w2cZGe9TiXBWUqWEEig0bmQQVx2ZWWO\\nv0og0X\\/iROXAcp6Z9WGpIc6FhVgJd\\/4bNlTR+A\\/lWQwFt1b6l03xdsyaIyIWi9xr\\nxsb2sLNWP56A\\/5TWTpOkfDbGCQrqHvukWSHlYFOzgQa0ZtMnV71ykH0CgYEAwSSY\\nqFfdAWrvVZjp26Yf\\/jnZavLCAC5hmho7eX5isCVcX86MHqpEYAFCecZN2dFFoPqI\\nyzHzgb9N6Z01YUEKqrknO3tA6JYJ9ojaMF8GZWvUtPzN41ksnD4MwETBEd4bUaH1\\n\\/pAcw\\/+\\/oYsh4BwkKnVHkNw36c+WmNoaX1FWqIsCgYBYw\\/IMnLa3drm3CIAa32iU\\nLRotP4qGaAMXpncsMiPage6CrFVhiuoZ1SFNbv189q8zBm4PxQgklLOj8B33HDQ\\/\\nlnN2n1WyTIyEuGA\\/qMdkoPB+TuFf1A5EzzZ0uR5WLlWa5nbEaLdNoYtBK1P5n4Kp\\nw7uYnRex6DGobt2mD+10cQKBgGVQlyune20k9QsHvZTU3e9z1RL+6LlDmztFC3G9\\n1HLmBkDTjjj\\/xAJAZuiOF4Rs\\/INnKJ6+QygKfApRxxCPF9NacLQJAZGAMxW50AqT\\nrj1BhUCzZCUgQABtpC6vYj\\/HLLlzpiC05AIEhDdvToPK\\/0WuY64fds0VccAYmMDr\\nX\\/PlAoGAS6UhbCm5TWZhtL\\/hdprOfar3QkXwZ5xvaykB90XgIps5CwUGCCsvwQf2\\nDvVny8gKbM\\/OenwHnTlwRTEj5qdeAM40oj\\/mwCDc6kpV1lJXrW2R5mCH9zgbNFla\\nW0iKCBUAm5xZgU\\/YskMsCBMNmA8A5ndRWGFEFE+VGDVPaRie0ro=\\n-----END RSA PRIVATE KEY-----';
            updateStatus();
        };
    </script>
</body>
</html>
