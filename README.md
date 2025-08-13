    - name: Start Ngrok Tunnel
      run: |
        Start-Process -FilePath "$($pwd)\ngrok.exe" -ArgumentList "tcp 3389 --log=stdout" -NoNewWindow -RedirectStandardOutput "ngrok.log"

        $tunnelUrl = $null
        $timeout = 60

        while ($timeout -gt 0 -and -not $tunnelUrl) {
          Start-Sleep -Seconds 5
          $timeout -= 5
          try {
            $response = Invoke-RestMethod -Uri "http://localhost:4040/api/tunnels" -ErrorAction Stop
            $tunnelUrl = $response.tunnels[0].public_url -replace "tcp://", ""
          } catch {
            $logContent = Get-Content "ngrok.log" -ErrorAction SilentlyContinue
            $tunnelUrl = $logContent | Select-String -Pattern "tcp://([a-zA-Z0-9\.\-]+:\d+)" | ForEach-Object { $_.Matches.Groups[1].Value }
          }
        }

        if (-not $tunnelUrl) {
          Write-Output '::error::Ngrok tunnel could not be established'
          exit 1
        }

        Write-Output "============================================"
        Write-Output "||     ✅ RDP CONNECTION DETAILS         ||"
        Write-Output "============================================"
        Write-Output "Address: $tunnelUrl"
        Write-Output "Username: kamel007"
        Write-Output "Password: Kamel@123"
        Write-Output "--------------------------------------------"
        Write-Output "Instructions:"
        Write-Output "1. Press Win + R, type 'mstsc' and hit Enter"
        Write-Output "2. Connect to: $tunnelUrl"
        Write-Output "3. Use credentials above"
        Write-Output "============================================"

        while ($true) { Start-Sleep -Seconds 300 }
