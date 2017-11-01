This blog will guide you how to setup Mongo DB with Authentication using Docker. So the data will be protected after exposing the db over network also and anyone can access the database from any where over internet with valid credentials.

Pre Requisites :- 
	1. You should having Docker installed in system where you are going to install Mongo DB 
	2. Download the Mongo DB setup file for docker from :- ""
	3. Open terminal verify once that your system has Docker install :- "docker -v"

Step1: Using terminal open downloaded folder "docker-mongo-auth" using command :- cd docker-mongo-auth

Step2: Editing Commands for Authorization configuration in Docker file
	Open Dockerfile using command :- "nano docker_fileName(In Downloaded folder its namely Dockerfile)"
	Change Required Component in Dockerfile:-

- In Dockerfile
  ```
	{
		FROM mongo	
			
			# Auth Configuration. Modify as needed. 
			# These environment variables can also be specified through command line or docker-compose configuration
			ENV AUTH yes
			ENV MONGODB_ADMIN_USER root // In place of root write your own admin username
			ENV MONGODB_ADMIN_PASS password // In place of password write your own admin password
			ENV MONGODB_APPLICATION_DATABASE your_db // In place of your_db write your own database name
			ENV MONGODB_APPLICATION_USER user // In place of user write general user's username
			ENV MONGODB_APPLICATION_PASS password // In place of password write general user's password
			EXPOSE 27017 27017
			ADD run.sh /run.sh
			ADD set_mongodb_password.sh /set_mongodb_password.sh
			RUN chmod +x /run.sh
			RUN chmod +x /set_mongodb_password.sh
			CMD ["/run.sh"]
	}
  ```

Step3: Editing Commands for creating Credentials in set_mongo_password.sh file
	Open set_mongo_password.sh file using command :- "nano set_mongo_password.sh" 
	Change Required component in set_mongo_password.sh:- 

- In set_mongo_password.sh
  ```
	{
		# Admin User
		MONGODB_ADMIN_USER=${MONGODB_ADMIN_USER:-"admin_username"}
		MONGODB_ADMIN_PASS=${MONGODB_ADMIN_PASS:-"admin_password"}
		# Application Database User
		MONGODB_APPLICATION_DATABASE=${MONGODB_APPLICATION_DATABASE:-"db_name"}
		MONGODB_APPLICATION_USER=${MONGODB_APPLICATION_USER:-"general_username"}
		MONGODB_APPLICATION_PASS=${MONGODB_APPLICATION_PASS:-"general_password"}
		# Wait for MongoDB to boot
		RET=1
		while [[ RET -ne 0 ]]; do
		echo "=> Waiting for confirmation of MongoDB service startup..."
		sleep 5
		mongo admin --eval "help" >/dev/null 2>&1
		RET=$?
		done
		# Create the admin user
		echo "=> Creating admin user with a password in MongoDB"
		mongo admin --eval "db.createUser({user: '$MONGODB_ADMIN_USER', pwd: '$MONGODB_ADMIN_PASS', roles:[{role:'root',db:'admin'}]});"
		sleep 3
		# If we've defined the MONGODB_APPLICATION_DATABASE environment variable and it's a different database
		# than admin, then create the user for that database.
		# First it authenticates to Mongo using the admin user it created above.
		# Then it switches to the REST API database and runs the createUser command 
		# to actually create the user and assign it to the database.
		if [ "$MONGODB_APPLICATION_DATABASE" != "admin" ]; then
		echo "=> Creating a ${MONGODB_APPLICATION_DATABASE} database user with a password in MongoDB"
		mongo admin -u $MONGODB_ADMIN_USER -p $MONGODB_ADMIN_PASS << EOF
		echo "Using $MONGODB_APPLICATION_DATABASE database"
		use $MONGODB_APPLICATION_DATABASE
		db.createUser({user: '$MONGODB_APPLICATION_USER', pwd: '$MONGODB_APPLICATION_PASS', roles:[{role:'dbOwner', db:'$MONGODB_APPLICATION_DATABASE'}]})
		EOF
		fi
		sleep 1
		# If everything went well, add a file as a flag so we know in the future to not re-create the
		# users if we're recreating the container (provided we're using some persistent storage)
		touch /data/db/.mongodb_password_set
		echo "MongoDB configured successfully. You may now connect to the DB."
	}
  ```

Step4: Building docker image and 
	Build image with the command :- "docker build -t docker_image_name ." //Docker_image_name is your image name
	Check that your docker image is created or not with the command :- "docker images" //You will get all images
	Run the images with command :- "docker run -d -p PORT_EXPOSED:27017 docker_image_name"
	Check your docker is running with the command :- "docker ps" //You will get all docker running image list 

Step5: Now connect from you mongoclient and check the connection via providing proper credentials.
