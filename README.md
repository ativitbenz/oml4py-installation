# oml4py-installation
Original document: https://docs.oracle.com/en/database/oracle/machine-learning/oml4py/1/mlpug/oracle-machine-learning-python.html#GUID-D00976CA-3663-4F32-A6A2-B6BF5A843ADC

`note`: OML4Py on premises runs on 64-bit platforms only.

Both client and server on-premises components are supported on the Linux platforms listed in the table below.

On-Premises OML4Py Platform Requirements
Operating System and Hardware Platform	Description
  - Oracle Linux x86-64 7.x 64-bit Oracle Linux Release 7
  - Oracle Linux x86-64 8.x 64-bit Oracle Linux Release 8

On-Premises OML4Py Configuration Requirements and Server Support Matrix
  - Oracle Machine Learning for Python Version	Python Version	On-Premises Oracle Database Release 1.0	3.9.5	19c, 21c

setp1: Build and Install Python for Linux for On-Premises Databases
Go to the Python website and download the Gzipped source tarball. The downloaded file name is Python-3.9.5.tgz
<clipboard-copy for="blob-path" class="btn btn-sm BtnGroup-item">
  Copy path
</clipboard-copy>
<div id="blob-path">src/index.js</div>
wget https://www.python.org/ftp/python/3.9.5/Python-3.9.5.tgz 

var copy = function(target) {
    var textArea = document.createElement('textarea')
    textArea.setAttribute('style','width:1px;border:0;opacity:0;')
    document.body.appendChild(textArea)
    textArea.value = target.innerHTML
    textArea.select()
    document.execCommand('copy')
    document.body.removeChild(textArea)
}

var pres = document.querySelectorAll(".comment-body > pre")
pres.forEach(function(pre){
  var button = document.createElement("button")
  button.className = "btn btn-sm"
  button.innerHTML = "copy"
  pre.parentNode.insertBefore(button, pre)
  button.addEventListener('click', function(e){
    e.preventDefault()
    copy(pre.childNodes[0])
  })
})
