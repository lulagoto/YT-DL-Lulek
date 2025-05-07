Download:

    https://www.gyan.dev/ffmpeg/builds/

ffmpeg-git-essentials.7z
or 
ffmpeg-git-full.7z

and extract files to folder ffmpeg:

    ffmpeg \ 
    > ffmpeg.exe
    > ffplay.exe
    > ffprobe.exe

paste yt links to list.txt
open RUN.bat

######################################
FILES:
######################################

RUN.bat 
run the all proces. ( simple script to run a .ps1 script code )

    @echo off
    powershell -ExecutionPolicy Bypass -File ".\scr\skrypt.ps1"
    pause  

skrypt.ps1
Have options to dl mp3 / mp3 and save mp4 / only mp4


FEATURES:
- Adding export bookmarks from browser, show only youtube links and check links/names what u dont want to download
- Print list links with error
- Analyse link list and show some info before starting dl

skrypt.ps1

    # Nagłówek
    function Show-Header {
        Write-Host "`n==================================================" -ForegroundColor DarkGray
        Write-Host "========= YT-DL skryptowany przez Lulka =========" -ForegroundColor White 
        Write-Host "==================================================`n" -ForegroundColor DarkGray
    
    Write-Host "Fast Info:" -ForegroundColor Yellow
    Write-Host "Program sciaga pliki w formacie z yt, osobno dzwiek osobno obraz dla MP4."
    Write-Host "Przy samym MP3 sciaga plik audio MP4 i konwertuje nastepnie usuwa."
    Write-Host "Przy samym MP4 sciagniete pliki MP4 (wymienione wyzej) laczy w jeden reszte usuwa."
    Write-Host "Przy wyborze opcji 2 zostaja smieci w pozostalosciach wymienionych wyzej -"
    Write-Host "-zamieszczone skrypty usuwaja smieci i zachowuja dobre MP4 opcji 2!`n"
    }
    
    # Opcje
    function Show-Options {
        Write-Host "==================================================" -ForegroundColor DarkGray
        Write-Host "=============== DOSTEPNE OPCJE: ==================" -ForegroundColor White 
        Write-Host "==================================================`n" -ForegroundColor DarkGray
    
    Write-Host "1. Pobierz jako MP3" -ForegroundColor Cyan
    Write-Host "   * Konwersja do wysokiej jakości MP3 (320kbps)`n"
    
    Write-Host "2. Pobierz jako MP3 i zachowaj MP4" -ForegroundColor Cyan
    Write-Host "   * Zachowuje oryginalne wideo + tworzy MP3`n"
    
    Write-Host "3. Pobierz tylko MP4" -ForegroundColor Cyan
    Write-Host "   * Tylko wideo w jakosci do 720p (chyba)`n"
    }
    
    # Opcje i headery
    Show-Header
    Show-Options
    
    # Wybór opcji z sprawdzaniem
    do {
        $choice = Read-Host "Wybierz opcje (1-3) / q - zakoncz"
        if ($choice -eq 'q' -or $choice -eq 'Q') {
            Write-Host "Zamykanie programu..." -ForegroundColor Yellow
            exit
        }
        elseif ($choice -notin '1','2','3') {
            Write-Host "Nieprawidłowy wybór. Podaj liczbę 1-3 lub 'q' aby wyjść" -ForegroundColor Red
        }
    } while ($choice -notin '1','2','3')
    
    # Konfiguracja
    $YTDLP_PATH = ".\scr\yt-dlp.exe"
    $FFMPEG_PATH = ".\ffmpeg\"
    $INPUT_FILE = ".\list.txt"
    $WORKING_DIR = ".\working"
    $MP3_DIR = ".\Downloads\mp3"
    $MP4_DIR = ".\Downloads\mp4"
    
    # Tworzenie folderów jeżeli nie ma
    if (-not (Test-Path $WORKING_DIR)) { New-Item -ItemType Directory -Path $WORKING_DIR | Out-Null }
    if (-not (Test-Path $MP3_DIR)) { New-Item -ItemType Directory -Path $MP3_DIR | Out-Null }
    if (-not (Test-Path $MP4_DIR)) { New-Item -ItemType Directory -Path $MP4_DIR | Out-Null }
    
    # Komenda od której wszystko się zaczeło - output do working
    $base_command = "& `"$YTDLP_PATH`" --ffmpeg-location `"$FFMPEG_PATH`" -a `"$INPUT_FILE`""
    $output_template = "$WORKING_DIR\%(title)s-%(id)s.%(ext)s"
    
    switch ($choice) {
        "1" {
            # Pobierz jako MP3
            $command = "$base_command -x --audio-format mp3 --audio-quality 0 -o `"$output_template`""
            Write-Host "`nOpcja: Pobierz jako MP3`n"
        }
        "2" {
            # Pobierz jako MP3 i zachowaj MP4
            $command = "$base_command -x --audio-format mp3 --audio-quality 0 -k -o `"$output_template`""
            Write-Host "`nOpcja: Pobierz jako MP3 i zachowaj MP4`n"
        }
        "3" {
            # Pobierz tylko MP4
            $command = "$base_command -f `"bestvideo[height<=720]+bestaudio/best[height<=720]`" -o `"$output_template`""
            Write-Host "`nOpcja: Pobierz tylko MP4`n"
        }
        default {
            Write-Host "Coś jakaś lipa, error."
            exit
        }
    }
    
    # Pobieranie
    Write-Host "Komenda:`n$command`n" -ForegroundColor Cyan
    Invoke-Expression $command
    
    # Po zakończeniu pobierania
    Write-Host "`nPrzetwarzanie itp..." -ForegroundColor Cyan
    Start-Sleep -Seconds 3  # timeout do startu
    
    # Pliki niepotrzebne ( pozostałości  zopcji nr 2 )
    $temp_files = Get-ChildItem $WORKING_DIR -File | Where-Object { 
        $_.Name -match '\.f\d+\.mp4$' -or $_.Extension -eq '.m4a' -or $_.Name -match '\.temp\.'
    }
    if ($temp_files) {
        $temp_files | Remove-Item -Force
        Write-Host "Usunieto $($temp_files.Count) plikow do usuniecia." -ForegroundColor Yellow
    }
    
    # Przenoszenie plików
    switch ($choice) {
        "1" {
            # Tylko MP3
            $mp3_files = Get-ChildItem $WORKING_DIR -Filter "*.mp3" -File
            if ($mp3_files) {
                $mp3_files | Move-Item -Destination $MP3_DIR -Force
                Write-Host "Przeniesiono $($mp3_files.Count) plikow MP3 do folderu \Downloads\mp3" -ForegroundColor Green
            }
        }
        "2" {
            # Przenieś MP3 i MP4
            $mp3_files = Get-ChildItem $WORKING_DIR -Filter "*.mp3" -File
            $mp4_files = Get-ChildItem $WORKING_DIR -Filter "*.mp4" -File
            
            if ($mp3_files) {
                $mp3_files | Move-Item -Destination $MP3_DIR -Force
                Write-Host "Przeniesiono $($mp3_files.Count) plikow MP3 do folderu \Downloads\mp3" -ForegroundColor Green
            }
            
            if ($mp4_files) {
                $mp4_files | Move-Item -Destination $MP4_DIR -Force
                Write-Host "Przeniesiono $($mp4_files.Count) plikow MP4 do folderu \Downloads\mp4" -ForegroundColor Green
            }
        }
        "3" {
            # Tylko MP4
            $mp4_files = Get-ChildItem $WORKING_DIR -Filter "*.mp4" -File
            if ($mp4_files) {
                $mp4_files | Move-Item -Destination $MP4_DIR -Force
                Write-Host "Przeniesiono $($mp4_files.Count) plikow MP4 do folderu \Downloads\mp4" -ForegroundColor Green
            }
        }
    }
    
    # Clean folderu working w razie W
    $remaining_files = Get-ChildItem $WORKING_DIR -File
    if ($remaining_files) {
        $remaining_files | Remove-Item -Force
        Write-Host "Usunieto $($remaining_files.Count) pozostalych smieci." -ForegroundColor DarkYellow
    }
    
    # Enter na zakończenie
    if ($Host.Name -eq "ConsoleHost") {
        Write-Host "`nNacisnij Enter, aby zakonczyc..." -ForegroundColor Yellow
        Read-Host
    }

