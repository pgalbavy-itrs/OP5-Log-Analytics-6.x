# Manual update of OP5 from version 6.1.1 to 6.1.2 #

The OP5 Log Anlytics 6.1.1 update should be done by copying new versions of files to the appropriate directories. The source installation directory is: /root/pkg_6.1.2*

## Data node update ##

1. Go to installation directory that contain OP5 Log Analytics 6.1.2 files:

		cd /root/pkg_6.1.2

1. Stop the Elasticsearch service
		
		systemctl stop elasticsearch

1. Backup the *elasticsearch-auth* plugin

		cp –rf /usr/share/elasticsearch/plugins/elasticsearch-auth /root/backup/

1. Copy the new version of the plugin from the installation directory:

		cp -rf elastisearch/elasticearch-auth /usr/share/elasticsearch/plugins/

1. Grant the appropriate permissions for directories:

		chown -R elasticseach:elasticsearch /usr/share/elasticsearch

1. Start the Elasticsearch service

		systemctl start elasticsearch

## Client Node update ##

1. Check if the following RPM packages have been installed:

		yum install fontconfig freetype freetype-devel fontconfig-devel libstdc++ urw-fonts net-tools ImageMagick ghostscript poppler-utils

1. Stop the Kibana and Alert services:

		systemctl stop kibana
		systemctl stop alert

1. Delete the contents of the directory */usr/share/kibana/optimize/bundles/*

		rm -rf /usr/share/kibana/optimize/bundles/*

1. Delete the contents of the directory */usr/share/kibana/plugins*

		rm -rf /usr/share/kibana/plugins/*

1. Backup current Alert rules folder

		cp -pr /opt/alert/rules /root/backup/alert/rules

1. Delete old version of Alert plugin

		rm -rf /opt/alert

1. Delete old version of AI plugin

		rm -rf /opt/ai

1. Copy the Kibana plugins from the installation directory

		cp -rf kibana/plugins/* /usr/share/kibana/plugins/

1. Copy the Alert plugin from the installation directory

		/bin/cp -rf alert /opt/alert

1. Copy Alert rules folder

		cp -pr /root/backup/alert/rules/* /opt/alert/rules/

1. Copy the Alert plugin from the installation directory

		 /bin/cp -rf ai /opt/ai

1. Unpack the node.js modules

		tar -xf /usr/share/kibana/plugins/node_modules.tar -C /usr/share/kibana/plugins/

1. Delete the unnecessary tar.gz archive

		/bin/rm -rf /usr/share/kibana/plugins/node_modules.tar

1. Give the right permissions to the Kibana directory

		chown -R kibana:kibana /usr/share/kibana

1. Perform the start of Kibana and Alert

		systemctl start kibana
		systemctl start alert
