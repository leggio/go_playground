package main

import (
	"fmt"
	"github.com/akamensky/argparse"
	"github.com/pkg/sftp"
	"golang.org/x/crypto/ssh"
	"io"
	"os"
	"path/filepath"
	"strconv"
	"strings"
	"time"
)


type connection_details struct {
	ftpHost string
	ftpPort int
	ftpUser string
	ftpPass string
	ftpDownloadPath string
}


func sftp_connect(connDetails connection_details) *sftp.Client {
	addr := connDetails.ftpHost + ":" + strconv.Itoa(connDetails.ftpPort)
	config := &ssh.ClientConfig{
		User: connDetails.ftpUser,
		HostKeyCallback: ssh.InsecureIgnoreHostKey(),
		Auth: []ssh.AuthMethod{
			ssh.Password(connDetails.ftpPass),
		},
	}

	config.Ciphers = append(config.Ciphers, "3des-cbc")

	conn, err := ssh.Dial("tcp", addr, config)
	if err != nil {
		panic("Failed to connect to SFTP site: " + err.Error())
	}

	fmt.Println("Successfully connected to SFTP site")

	client, err := sftp.NewClient(conn)
	if err != nil {
		panic("Failed to create SFTP client: " + err.Error())
	}

	return client
}

func get_files(client *sftp.Client, ftpDownloadPath string) {
	target_time := time.Now().Local().AddDate(0, 0, -10)
	fmt.Println(target_time)

	target_files := []string{}

	files := client.Walk(ftpDownloadPath)

	for files.Step() {
		if files.Err() != nil {
			continue
		}
		mod := files.Stat().ModTime()
		if mod.After(target_time) {
			target_files = append(target_files, files.Path())
		}
	}

	temp_directory := "temp"
	_ = os.Mkdir(temp_directory, 0700)
	_ = os.Chdir(temp_directory)

	for _, file := range target_files {
		if strings.HasSuffix(file, ".csv") {
			src, err := client.Open(file)
			if err != nil {
				panic("Could not open file: " + err.Error())
			}
			defer src.Close()

			dst, err := os.Create(filepath.Base(file))
			if err != nil {
				panic("Could not create file: " + err.Error())
			}
			defer dst.Close()

			bytes, err := io.Copy(dst, src)
			if err != nil {
				panic("Could not copy file: " + err.Error())
			}
			fmt.Println(strconv.FormatInt(bytes, 10) + " bytes copied")

		}
	}

}

func main() {
	parser := argparse.NewParser("sftp_monitor", "Gets files from SFTP site")
	ftp_host := parser.String("c", "ftp_host", &argparse.Options{Required: false, Default: ""})
	ftp_port := parser.Int("d", "ftp_port", &argparse.Options{Required: false, Default: 22})
	ftp_user := parser.String("e", "ftp_user", &argparse.Options{Required: false, Default: ""})
	ftp_pass := parser.String("f", "ftp_pass", &argparse.Options{Required: false, Default: ""})
	ftp_download_path := parser.String("g", "ftp_download_path", &argparse.Options{Required: false, Default: ""})

	err := parser.Parse(os.Args)

	if err != nil {
		fmt.Println(parser.Usage(err))
	}

	connDetails := connection_details{ftpHost: *ftp_host, ftpPort: *ftp_port, ftpUser: *ftp_user, ftpPass: *ftp_pass,
		ftpDownloadPath: *ftp_download_path}

	fmt.Println(connDetails)
	client := sftp_connect(connDetails)

	get_files(client, connDetails.ftpDownloadPath)

	defer client.Close()
	fmt.Println("Closed SFTP client connection")
}
