WINEARCH ?= win32

# versions
PYTHON_VERSION       := 3.6.8
PYTHON_VERSION_DL    := 3.6.8
PYTHON_VERSION_MAJOR := $(word 1,$(subst ., ,${PYTHON_VERSION})).$(word 2,$(subst ., ,${PYTHON_VERSION}))
PYTHON_VERSION_SHORT := $(subst .,,${PYTHON_VERSION_MAJOR})
CXFREEZE_VERSION     := 6.1
LXML_VERSION         := 4.2.0
BITARRAY_VERSION     := 0.8.3

# installers
DOWNLOAD_DIR            ?= dl
PYTHON_INSTALLER-win32  := ${DOWNLOAD_DIR}/python-${PYTHON_VERSION_DL}.exe
PYTHON_INSTALLER-win64  := ${DOWNLOAD_DIR}/python-${PYTHON_VERSION_DL}-amd64.exe
PYTHON_INSTALLER        := ${PYTHON_INSTALLER-${WINEARCH}}
CXFREEZE_WHEEL-win32    := ${DOWNLOAD_DIR}/cx_Freeze-${CXFREEZE_VERSION}-cp${PYTHON_VERSION_SHORT}-cp${PYTHON_VERSION_SHORT}m-${WINEARCH}.whl
CXFREEZE_WHEEL-win64    := ${DOWNLOAD_DIR}/cx_Freeze-${CXFREEZE_VERSION}-cp${PYTHON_VERSION_SHORT}-cp${PYTHON_VERSION_SHORT}m-win_amd64.whl
CXFREEZE_WHEEL          := ${CXFREEZE_WHEEL-${WINEARCH}}
LXML_WHEEL-win32        := ${DOWNLOAD_DIR}/lxml-${LXML_VERSION}-cp${PYTHON_VERSION_SHORT}-none-${WINEARCH}.whl
LXML_WHEEL-win64        := ${DOWNLOAD_DIR}/lxml-${LXML_VERSION}-cp${PYTHON_VERSION_SHORT}-none-win_amd64.whl
LXML_WHEEL              := ${LXML_WHEEL-${WINEARCH}}
BITARRAY_WHEEL-win32    := ${DOWNLOAD_DIR}/bitarray-${BITARRAY_VERSION}-cp${PYTHON_VERSION_SHORT}-cp${PYTHON_VERSION_SHORT}m-${WINEARCH}.whl
BITARRAY_WHEEL-win64    := ${DOWNLOAD_DIR}/bitarray-${BITARRAY_VERSION}-cp${PYTHON_VERSION_SHORT}-cp${PYTHON_VERSION_SHORT}m-win_amd64.whl
BITARRAY_WHEEL          := ${BITARRAY_WHEEL-${WINEARCH}}
7ZIP_INSTALLER          := ${DOWNLOAD_DIR}/7z938.msi

# env
CURL     := curl -s -f -L -m 600 --connect-timeout 30 --retry 10 --retry-max-time 5
WINE_DIR := /tmp/$(subst /,_,${CURDIR})/wine-${WINEARCH}
WINE     := mkdir -p $(dir ${WINE_DIR}) && WINEPREFIX=${WINE_DIR} WINEARCH=${WINEARCH} WINEDLLOVERRIDES="mscoree,mshtml=" WINEDEBUG=-all wine

# installed files
PYTHON_INSTALLED   := ${WINE_DIR}/drive_c/Python${PYTHON_VERSION_SHORT}/python.exe
CXFREEZE_INSTALLED := ${WINE_DIR}/drive_c/Python${PYTHON_VERSION_SHORT}/Scripts/cxfreeze
PIP_INSTALLED      := ${WINE_DIR}/drive_c/Python${PYTHON_VERSION_SHORT}/Scripts/pip.exe
LXML_INSTALLED     := ${WINE_DIR}/drive_c/Python${PYTHON_VERSION_SHORT}/Lib/site-packages/lxml/__init__.py
BITARRAY_INSTALLED := ${WINE_DIR}/drive_c/Python${PYTHON_VERSION_SHORT}/Lib/site-packages/bitarray/__init__.py
7ZIP_INSTALLED     := ${WINE_DIR}/drive_c/7-Zip/7z.sfx

# temp dirs
TEST_LIB_DIR := ${WINE_DIR}/drive_c/music
OUTPUT_DIR   := sacad-${WINEARCH}


.PHONY: all test test_ut test_freeze fclean clean


all:: sacad_${WINEARCH}.7z sacad_${WINEARCH}.zip sacad_${WINEARCH}.exe

test:: test_ut test_freeze

print-%:
	@echo $*=$($*)


#
# End targets
#

sacad_${WINEARCH}.7z: ${OUTPUT_DIR}/sacad.exe ${OUTPUT_DIR}/sacad_r.exe ${OUTPUT_DIR}/LICENSE ${OUTPUT_DIR}/readme.txt ${OUTPUT_DIR}/python${PYTHON_VERSION_SHORT}.dll ${OUTPUT_DIR}/sqlite3.dll
	7z a -t7z -mx9 $@ ${OUTPUT_DIR} > /dev/null

sacad_${WINEARCH}.zip: ${OUTPUT_DIR}/sacad.exe ${OUTPUT_DIR}/sacad_r.exe ${OUTPUT_DIR}/LICENSE ${OUTPUT_DIR}/readme.txt ${OUTPUT_DIR}/python${PYTHON_VERSION_SHORT}.dll ${OUTPUT_DIR}/sqlite3.dll
	7z a -tzip -mx9 $@ ${OUTPUT_DIR} > /dev/null

sacad_${WINEARCH}.exe: ${7ZIP_INSTALLED} sacad_${WINEARCH}.7z
	cat $^ > $@


#
# Intermediate targets
#

${OUTPUT_DIR}/sacad.exe: ${CXFREEZE_INSTALLED} ${LXML_INSTALLED} ${BITARRAY_INSTALLED} .requirements
	mkdir -p $(dir $@)
	cd .. && ${WINE} ${PYTHON_INSTALLED} freeze.py build_exe -b win/${OUTPUT_DIR} 2> /dev/null > /dev/null
	touch $@

${OUTPUT_DIR}/sacad_r.exe: ${OUTPUT_DIR}/sacad.exe

${OUTPUT_DIR}/python${PYTHON_VERSION_SHORT}.dll: ${PYTHON_INSTALLED}
	mkdir -p $(dir $@)
	cp $(dir ${PYTHON_INSTALLED})$(notdir $@) $@

${OUTPUT_DIR}/sqlite3.dll: ${PYTHON_INSTALLED}
	mkdir -p $(dir $@)
	cp $(dir ${PYTHON_INSTALLED})DLLs/$(notdir $@) $@

${OUTPUT_DIR}/LICENSE:
	mkdir -p $(dir $@)
	cp ../LICENSE $@

${OUTPUT_DIR}/readme.txt:
	mkdir -p $(dir $@)
	echo 'Get source, help and latest version at: https://github.com/desbma/sacad' > $@


#
# Tests
#

test_ut: ${LXML_INSTALLED} ${BITARRAY_INSTALLED} .requirements
	cd .. && ${WINE} ${PYTHON_INSTALLED} setup.py test

test_freeze:: ${TEST_LIB_DIR}/test.jpg ${TEST_LIB_DIR}/test.png

${TEST_LIB_DIR}/test.jpg: ${OUTPUT_DIR}/sacad.exe ${OUTPUT_DIR}/python${PYTHON_VERSION_SHORT}.dll ${OUTPUT_DIR}/sqlite3.dll
	mkdir -p $(dir $@)
	${WINE} $< "neu" "neu 2" 1000 $@ && test -f $@

${TEST_LIB_DIR}/sample.ogg: ${DOWNLOAD_DIR}/sample.ogg
	mkdir -p $(dir $@)
	cp $< $@

${TEST_LIB_DIR}/test.png: ${TEST_LIB_DIR}/sample.ogg ${OUTPUT_DIR}/sacad_r.exe ${OUTPUT_DIR}/python${PYTHON_VERSION_SHORT}.dll ${OUTPUT_DIR}/sqlite3.dll
	${WINE} ${OUTPUT_DIR}/sacad_r.exe $(dir $@) 300 $(notdir $@) && test -f $@


#
# Installations
#

.requirements: ${PIP_INSTALLED}
	cd .. && ${WINE} ${PYTHON_INSTALLED} -m pip -qqq install -r requirements.txt
	touch $@

${PYTHON_INSTALLED}: ${PYTHON_INSTALLER}
	${WINE} $< /quiet InstallAllUsers=1 TargetDir=$(dir ${PYTHON_INSTALLED})
	touch $@

${CXFREEZE_INSTALLED}: ${CXFREEZE_WHEEL} ${PIP_INSTALLED} ${PYTHON_INSTALLED}
	${WINE} ${PYTHON_INSTALLED} -m pip -qqq install $<
	touch $@

${PIP_INSTALLED}: ${PYTHON_INSTALLED}
	${WINE} ${PYTHON_INSTALLED} -m pip -qqq install -U pip
	touch $@

${LXML_INSTALLED}: ${LXML_WHEEL} ${PIP_INSTALLED} ${PYTHON_INSTALLED}
	${WINE} ${PYTHON_INSTALLED} -m pip -qqq install $<
	touch $@

${BITARRAY_INSTALLED}: ${BITARRAY_WHEEL} ${PIP_INSTALLED} ${PYTHON_INSTALLED}
	${WINE} ${PYTHON_INSTALLED} -m pip -qqq install $<
	touch $@

${7ZIP_INSTALLED}: ${7ZIP_INSTALLER}
	${WINE} msiexec /i $< /qb INSTALLDIR='c:\7-Zip'
	touch $@


#
# Downloads
#

${PYTHON_INSTALLER}:
	mkdir -p $(dir $@)
	${CURL} https://www.python.org/ftp/python/${PYTHON_VERSION}/$(notdir $@) > $@

${CXFREEZE_WHEEL-win32}:
	mkdir -p $(dir $@)
	${CURL} https://files.pythonhosted.org/packages/ea/2b/076f152fdaff10a08ca4012713532b6d2606c1995ee85812d271135f90e2/$(notdir $@) > $@

${CXFREEZE_WHEEL-win64}:
	mkdir -p $(dir $@)
	${CURL} https://files.pythonhosted.org/packages/50/dc/ad7111fc626a24504e468382c431698e4dcd60c944b9e9adf2187d073865/$(notdir $@) > $@

${LXML_WHEEL-win32}:
	mkdir -p $(dir $@)
	${CURL} https://www.dropbox.com/s/hqea07bpb3sij78/$(notdir $@) > $@

${LXML_WHEEL-win64}:
	mkdir -p $(dir $@)
	${CURL} https://www.dropbox.com/s/1p0nt3z49enp7x0/$(notdir $@) > $@

${BITARRAY_WHEEL-win32}:
	mkdir -p $(dir $@)
	${CURL} https://www.dropbox.com/s/wiwhdyu4p2fjdl7/$(notdir $@) > $@

${BITARRAY_WHEEL-win64}:
	mkdir -p $(dir $@)
	${CURL} https://www.dropbox.com/s/64nl2ug32z4yf8v/$(notdir $@) > $@

${7ZIP_INSTALLER}:
	mkdir -p $(dir $@)
	${CURL} http://www.7-zip.org/a/$(notdir $@) > $@.unckecked
	echo "673377a03f91ff0f7f384794934b767c3389e50d7dc308f7c73f4633121518b3e4af8d33cf282398a710577e12fbb963152591c3e29b1092c79530bb2d8aa882  $@.unckecked" | sha512sum --status -c - && mv $@.unckecked $@

${DOWNLOAD_DIR}/sample.ogg:
	mkdir -p $(dir $@)
	${CURL} https://www.dropbox.com/s/zg80jd35ie3aury/Hydrate-Kenny_Beltrey.ogg > $@


#
# Clean
#

fclean:
	-rm -Rf ${WINE_DIR}
	-rm -Rf ${OUTPUT_DIR} .requirements
	-rm -f sacad_${WINEARCH}.7z sacad_${WINEARCH}.zip sacad_${WINEARCH}.exe

clean: fclean
	-rm -Rf ${DOWNLOAD_DIR}
