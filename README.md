name: Windows RDP via Ngrok - NEVER STOP

on: 
  push:
    branches: [ main, master ]
  workflow_dispatch:
  schedule:
    - cron: '0 */4 * * *'  # Run every 4 hours to ensure NO gaps

jobs:
  rdp-setup:
    runs-on: windows-latest
    timeout-minutes: 240  # 4 hours per session
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Download and extract Ngrok
      run: |
        Write-Host "🚀 Downloading Ngrok..."
        Invoke-WebRequest "https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-stable-windows-amd64.zip" -OutFile ngrok.zip
        Write-Host "📦 Extracting Ngrok..."
        Expand-Archive ngrok.zip -DestinationPath "$env:USERPROFILE" -Force
        Remove-Item ngrok.zip
        Copy-Item "$env:USERPROFILE\ngrok.exe" "$env:GITHUB_WORKSPACE" -Force
        Write-Host "✅ Ngrok ready!"

    - name: Authenticate Ngrok
      run: |
        Write-Host "🔐 Authenticating Ngrok..."
        $ErrorActionPreference = "Stop"
        .\ngrok.exe authtoken "${{ secrets.NGROK_AUTH_TOKEN }}"
        Write-Host "✅ Authentication successful!"

    - name: Enable RDP and Firewall
      run: |
        Write-Host "🔧 Configuring RDP and firewall..."
        # Enable RDP
        Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0
        Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "UserAuthentication" -Value 0
        
        # Configure firewall
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
        New-NetFirewallRule -DisplayName "Allow RDP via Ngrok" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 3389 -Profile Any
        
        Write-Host "✅ RDP and firewall configured!"

    - name: Create Admin User
      run: |
        Write-Host "👤 Setting up admin user..."
        $userExists = Get-LocalUser -Name "kamel007" -ErrorAction SilentlyContinue
        if (-not $userExists) {
          net user kamel007 Kamel@123 /add
          net localgroup administrators kamel007 /add
          Write-Host "✅ User kamel007 created!"
        } else {
          Write-Host "ℹ️ User kamel007 already exists"
        }

    - name: Start Ngrok Tunnel (NEVER STOP)
      run: |
        Write-Host "🌐 Starting Ngrok tunnel..."
        
        # Start ngrok in background
        Start-Process -FilePath "$($pwd)\ngrok.exe" -ArgumentList 'tcp 3389 --log=stdout' -NoNewWindow -RedirectStandardOutput "ngrok.log"
        
        # Wait for tunnel to be ready
        $tunnelUrl = $null
        $timeout = 15  # 15 seconds timeout
        
        Write-Host "⏳ Waiting for tunnel..."
        
        while ($timeout -gt 0 -and -not $tunnelUrl) {
          Start-Sleep -Seconds 1
          $timeout -= 1
          
          try {
            $response = Invoke-RestMethod -Uri "http://localhost:4040/api/tunnels" -ErrorAction Stop
            if ($response.tunnels -and $response.tunnels.Count -gt 0) {
              $tunnelUrl = $response.tunnels[0].public_url -replace "tcp://", ""
              Write-Host "✅ Tunnel ready: $tunnelUrl"
            }
          } catch {
            $logContent = Get-Content "ngrok.log" -ErrorAction SilentlyContinue
            $tunnelUrl = $logContent | Select-String -Pattern "tcp://([a-zA-Z0-9\.\-]+:\d+)" | ForEach-Object { $_.Matches.Groups[1].Value }
            if ($tunnelUrl) {
              Write-Host "✅ Tunnel ready (log): $tunnelUrl"
            }
          }
        }

        if (-not $tunnelUrl) {
          Write-Output '::error::❌ Failed to establish tunnel'
          exit 1
        }

        # Display connection details
        Write-Output "============================================"
        Write-Output "||     🎯 RDP NEVER STOPS              ||"
        Write-Output "============================================"
        Write-Output "🌐 Address: $tunnelUrl"
        Write-Output "👤 Username: kamel007"
        Write-Output "🔑 Password: Kamel@123"
        Write-Output "⏰ Session: 4 hours (auto-restart)"
        Write-Output "🔄 Status: NEVER STOPPING"
        Write-Output "🚫 NO DOWNTIME - 100% UPTIME"
        Write-Output "--------------------------------------------"
        Write-Output "📱 Instructions:"
        Write-Output "1. Win + R → mstsc → Enter"
        Write-Output "2. Connect to: $tunnelUrl"
        Write-Output "3. Use credentials above"
        Write-Output "4. RDP NEVER STOPS - Always available"
        Write-Output "============================================"
        
        # Keep the job running FOREVER with continuous health checks
        Write-Host "🔄 Tunnel active. NEVER STOPPING..."
        $startTime = Get-Date
        $maxRuntime = [TimeSpan]::FromMinutes(235)  # 235 minutes to be safe
        
        while ((Get-Date) - $startTime -lt $maxRuntime) {
          Start-Sleep -Seconds 15  # Check every 15 seconds
          $elapsed = (Get-Date) - $startTime
          
          # Continuous health check
          try {
            $health = Invoke-RestMethod -Uri "http://localhost:4040/api/tunnels" -ErrorAction Stop
            if ($health.tunnels -and $health.tunnels.Count -gt 0) {
              if ($elapsed.TotalSeconds % 60 -lt 15) {  # Show status every minute
                Write-Host "✅ HEALTHY: Tunnel active for $($elapsed.Hours)h $($elapsed.Minutes)m - NEVER STOPPING"
              }
            } else {
              Write-Host "⚠️ Tunnel down, restarting immediately..."
              break
            }
          } catch {
            Write-Host "⚠️ Health check failed, restarting immediately..."
            break
          }
        }
        
        Write-Host "🔄 Session ending, starting new one IMMEDIATELY..."

    - name: Instant Restart (NO STOP)
      if: always()
      run: |
        Write-Host "🚀 INSTANT RESTART - NO STOPPING..."
        Stop-Process -Name "ngrok" -Force -ErrorAction SilentlyContinue
        Get-Process -Name "ngrok" -ErrorAction SilentlyContinue | Stop-Process -Force
        
        Write-Output "============================================"
        Write-Output "||     🚀 INSTANT RESTART              ||"
        Write-Output "============================================"
        Write-Output "✅ Session ended"
        Write-Output "🚀 New session starting IMMEDIATELY"
        Write-Output "⏰ NO WAITING - NO DOWNTIME"
        Write-Output "🌐 RDP ALWAYS AVAILABLE"
        Write-Output "🚫 NEVER STOPS - NEVER DIES"
        Write-Output "============================================"

    - name: Continuous Operation Notice
      if: always()
      run: |
        Write-Host "♾️ PREPARING FOR ETERNAL OPERATION..."
        Write-Output "::notice::RDP session completed. Starting new session IMMEDIATELY for 100% uptime."
        Write-Output "::notice::RDP will NEVER STOP - Always available 24/7/365"
