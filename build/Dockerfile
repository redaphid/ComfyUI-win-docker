FROM mcr.microsoft.com/windows/server:ltsc2022

EXPOSE 8188

ENV ARGS=

COPY ./res c:/res
COPY ./ComfyUI_windows_portable c:/app

RUN powershell Move-Item -Path c:\res\nvdll\System32\* -Destination C:\Windows\System32 ; \
    Move-Item -Path c:\res\nvdll\SysWOW64\* -Destination C:\Windows\SysWOW64 ; \
    Start-Process -wait -FilePath C:\res\vc_redist.x64.exe -ArgumentList '/install /passive /norestart' ; \
    Remove-Item -r c:\res

WORKDIR C:/app
ENTRYPOINT entry.bat
