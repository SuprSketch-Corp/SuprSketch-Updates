param(
    [Parameter(Mandatory=$true)]
    [string]$token,
    [Parameter(Mandatory=$false)]
    [string]$version = "1.0.1"
)

try {
    Write-Host "Setting up environment..."
    $env:GH_TOKEN = $token

    # Verify repository exists and is accessible
    $repoUrl = "https://api.github.com/repos/SuprSketch-Corp/SuprSketch.exe-Updates"
    $headers = @{
        "Authorization" = "Bearer $token"
        "Accept" = "application/vnd.github.v3+json"
    }
    
    Write-Host "Verifying repository access..."
    $response = Invoke-RestMethod -Uri $repoUrl -Headers $headers -Method Get -ErrorAction Stop
    
    Write-Host "Creating git tag..."
    git tag "v$version" -f
    git push origin "v$version" -f
    
    Write-Host "Building and publishing update..."
    npm run publish-update
    
    Write-Host "Update published successfully!"
} catch {
    Write-Host "Error occurred: $_" -ForegroundColor Red
} finally {
    Write-Host "Cleaning up..."
    Remove-Item Env:\GH_TOKEN -ErrorAction SilentlyContinue
}