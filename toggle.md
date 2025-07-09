sed -i "s|threshold: ['\\\"]\\?.*['\\\"]\\?|threshold: '${params.threshold}'|" ${kedaFile}
