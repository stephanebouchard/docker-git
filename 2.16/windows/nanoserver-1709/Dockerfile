# escape=`
FROM microsoft/windowsservercore:1709 AS build

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ENV GITPATH C:\\git

RUN $newPath = ('{0}\cmd;{1}' -f $env:GITPATH, $env:PATH); `
	Write-Host ('Updating PATH: {0}' -f $newPath); `
# Nano Server does not have "[Environment]::SetEnvironmentVariable()"
	setx /M PATH $newPath;
# doing this first to share cache across versions more aggressively

ENV GIT_VERSION 2.16
ENV GIT_VERSION_PATCH 2
ENV GIT_VERSION_BUILD 1
ENV GIT_CHECKSUM f4ac4e7d53d599d515e905824240cc2b82f3e2c294a872bb650e44b7e89cae8c
ENV GIT_CHECKSUM_TYPE sha256

RUN $filename = ('MinGit-{0}.{1}.{2}-64-bit.zip' -f $env:GIT_VERSION, $env:GIT_VERSION_PATCH, $env:GIT_VERSION_BUILD); `
    $filename = if ( $env:GIT_VERSION_BUILD -eq '1') { $filename -Replace '.1-', '-' } else { $filename }; `
    $url = ('https://github.com/git-for-windows/git/releases/download/v{0}.{1}.windows.{2}/{3}' -f $env:GIT_VERSION, $env:GIT_VERSION_PATCH, $env:GIT_VERSION_BUILD, $filename); `
    `
    Write-Host ('Downloading {0} ...' -f $url); `
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; `
    Invoke-WebRequest -Uri $url -OutFile $filename; `
    `
    Write-Host ('Verifying {0} ({1}) ...' -f $env:GIT_CHECKSUM_TYPE, $env:GIT_CHECKSUM); `
    if ((Get-FileHash $filename -Algorithm $env:GIT_CHECKSUM_TYPE).Hash -ne $env:GIT_CHECKSUM) { `
        Write-Host 'FAILED!'; `
        exit 1; `
    }; `
    `
    Write-Host 'Expanding ...'; `
    Expand-Archive $filename -DestinationPath $env:GITPATH; `
    `
    Write-Host 'Verifying install ...'; `
    Write-Host '  git --version'; git --version; `
    `
    Write-Host ('Removing {0} ...' -f $filename); `
    Remove-Item $filename -Force; `
    `
    Write-Host 'Complete.';


FROM microsoft/nanoserver:1709

COPY --from=build ["git", "C:\\git"]

RUN setx PATH "%PATH%;C:\git\cmd"

ENTRYPOINT ["git"]

CMD ["--version"]

WORKDIR "repos"
