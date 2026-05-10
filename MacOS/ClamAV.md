{
 "cells": [
  {
   "metadata": {},
   "cell_type": "markdown",
   "source": "Update Homebrew and Packages"
  },
  {
   "metadata": {},
   "cell_type": "code",
   "outputs": [],
   "execution_count": null,
   "source": [
    "brew update\n",
    "brew upgrade"
   ]
  },
  {
   "metadata": {},
   "cell_type": "markdown",
   "source": "Install Homebrew Formulae"
  },
  {
   "metadata": {
    "collapsed": true
   },
   "cell_type": "code",
   "outputs": [],
   "execution_count": null,
   "source": "brew install clamav"
  },
  {
   "metadata": {},
   "cell_type": "markdown",
   "source": "Setup Clamav"
  },
  {
   "metadata": {},
   "cell_type": "code",
   "source": [
    "sudo touch /var/log/freshclam.log\n",
    "sudo chmod 600 /var/log/freshclam.log\n",
    "sudo chown _clamav /var/log/freshclam.log\n",
    "\n",
    "sudo touch /var/log/clamd.log\n",
    "sudo chmod 600 /var/log/clamd.log\n",
    "sudo chown _clamav /var/log/clamd.log\n",
    "\n",
    "sudo chmod 750 /opt/homebrew/var/lib/clamav\n",
    "sudo chown _clamav /opt/homebrew/var/lib/clamav\n",
    "\n",
    "sudo mkdir -p /opt/homebrew/var/run/clamav\n",
    "sudo chown _clamav /opt/homebrew/var/run/clamav\n",
    "sudo chmod 750 /opt/homebrew/var/run/clamav\n",
    "\n",
    "rm -f /opt/homebrew/etc/clamav/freshclam.conf\n",
    "cat << EOF > /opt/homebrew/etc/clamav/freshclam.conf\n",
    "DatabaseDirectory /opt/homebrew/var/lib/clamav\n",
    "CVDCertsDirectory /opt/homebrew/etc/clamav/certs\n",
    "UpdateLogFile /var/log/freshclam.log\n",
    "LogFileMaxSize 2M\n",
    "LogTime yes\n",
    "LogSyslog yes\n",
    "LogRotate yes\n",
    "DatabaseOwner _clamav\n",
    "DatabaseMirror database.clamav.net\n",
    "EOF\n",
    "\n",
    "rm -f /opt/homebrew/etc/clamav/clamd.conf\n",
    "cat << EOF >> /opt/homebrew/etc/clamav/clamd.conf\n",
    "LogFile /var/log/clamd.log\n",
    "LogFileMaxSize 2M\n",
    "LogTime yes\n",
    "LogRotate yes\n",
    "DatabaseDirectory /opt/homebrew/var/lib/clamav\n",
    "CVDCertsDirectory /opt/homebrew/etc/clamav/certs\n",
    "MaxDirectoryRecursion 20\n",
    "User _clamav\n",
    "LocalSocket /opt/homebrew/var/run/clamav/clamd.sock\n",
    "LocalSocketGroup _clamav\n",
    "LocalSocketMode 660\n",
    "# VirusEvent\n",
    "EOF\n",
    "\n",
    "cat /opt/homebrew/etc/clamav/freshclam.conf\n",
    "cat /opt/homebrew/etc/clamav/clamd.conf"
   ],
   "outputs": [],
   "execution_count": null
  },
  {
   "metadata": {},
   "cell_type": "markdown",
   "source": "Prints Configuration"
  },
  {
   "metadata": {},
   "cell_type": "code",
   "outputs": [],
   "execution_count": null,
   "source": "cat /opt/homebrew/bin/clamconf"
  },
  {
   "metadata": {},
   "cell_type": "markdown",
   "source": "Runs freshclam"
  },
  {
   "metadata": {},
   "cell_type": "code",
   "outputs": [],
   "execution_count": null,
   "source": "sudo -u _clamav freshclam"
  },
  {
   "metadata": {},
   "cell_type": "markdown",
   "source": "Start Daemon"
  },
  {
   "metadata": {},
   "cell_type": "code",
   "outputs": [],
   "execution_count": null,
   "source": "clamd"
  },
  {
   "metadata": {},
   "cell_type": "markdown",
   "source": "Ad Hoc Scan"
  },
  {
   "metadata": {},
   "cell_type": "code",
   "outputs": [],
   "execution_count": null,
   "source": [
    "USER=<username>\n",
    "sudo clamscan -r --infected --log=/var/log/clamd.log /Users/${USER} --exclude=\"/Users/${USER}/Library/Application Support/virtualenv\""
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Kotlin",
   "language": "kotlin",
   "name": "kotlin"
  },
  "language_info": {
   "name": "kotlin",
   "version": "2.2.20-dev-4982",
   "mimetype": "text/x-kotlin",
   "file_extension": ".kt",
   "pygments_lexer": "kotlin",
   "codemirror_mode": "text/x-kotlin",
   "nbconvert_exporter": ""
  }
 },
 "nbformat": 4,
 "nbformat_minor": 0
}
