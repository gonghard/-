maven 常用插件用法
1.ftp 上传以及远程执行Linux命令(http://www.mojohaus.org/wagon-maven-plugin/)

		       <extensions>
					<extension>
						<groupId>org.apache.maven.wagon</groupId>
						<artifactId>wagon-ssh</artifactId>
						<version>2.8</version>
					</extension>
				</extensions>
				<plugins>
					<plugin>
						<groupId>org.codehaus.mojo</groupId>
						<artifactId>wagon-maven-plugin</artifactId>
						<version>1.0</version>
						<configuration>
		                    <!--执行wagon:upload-single即可以将需要上传的文件进行上传 -->
							<fromFile>target/test.war</fromFile>
							<url>scp://root:2013520@192.168.0.133/usr/local</url>
		                    <!--以下是将指定目录上产到指定目录用wagon:upload命令-->
		                    <!-- <fromDir>target/</fromDir>
							<url>scp://root:2013520@192.168.0.133</url>
							<toDir>/usr/local</toDir>-->
							<!-- 用wagon:sshexec执行以下命令 -->
							<commands>
								<!-- 杀死原来的进程 -->
								<command>pkill -f test.jar</command>
								<!-- 重新启动test.jar，程序的输出结果写到nohup.out文件中 -->
								<command>nohup java -jar /home/xxg/Desktop/test.jar >
									/home/xxg/Desktop/nohup.out 2>&amp;1 &amp;</command>
							</commands>
							<!-- 显示运行命令的输出结果 -->
							<displayCommandOutputs>true</displayCommandOutputs>
						</configuration>
					</plugin>

2.打war包插件		

				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-war-plugin</artifactId>
					<version>2.1.1</version>
					<configuration>
						<warName>test</warName>
						<failOnMissingWebXml>false</failOnMissingWebXml>
						<packagingExcludes>WEB-INF/web.xml</packagingExcludes>
					</configuration>
				</plugin>

3.tomcat7 部署以及运行


			<plugin>
				<groupId>org.apache.tomcat.maven</groupId>
				<artifactId>tomcat7-maven-plugin</artifactId>
				<version>2.2</version>
				<configuration>
                    <!--tomcat7:run 本地运行项目-->
					<port>9999</port><!--此端口表示本地项目运行的端口-->
                    <!--利用tomcat manager部署-->
					<url>http://192.168.0.11:8080/manager/text/</url>
					<server>tomcat-deploy</server>
                    <!--以下用户名和密码需要在tomca-user.xml文件中配置如下内容
	                <role rolename="tomcat"/>  
	                <role rolename="manager"/>      
	                <role rolename="manager-script"/>  
	                <role rolename="manager-status"/>  
	                <role rolename="manager-jmx"/>      
	                <role rolename="manager-gui"/>  
	                <role rolename="admin"/>  
	                <role rolename="admin-gui"/>   
	                <user username="admin" password="2013520" roles="admin,admin-gui,manager,manager-gui,manager-script,manager-status,manager-jmx"/>  
                    第一次部署用tomcat7:deploy命令，前提是tomcat必须先启动，然后tomcat webapps目录下manager目录还在
                    tomcat7:redeploy重新部署
                    tomcat7:undeploy删除部署
                    -->
					<username>用户名</username>
					<password>密码</password>
                    <!--项目部署访问路径-->
					<path>/${project.name}</path>
				</configuration>
			</plugin>

4.jetty运行maven web项目

			<plugin>
				<groupId>org.eclipse.jetty</groupId>
				<artifactId>jetty-maven-plugin</artifactId>
				<version>9.3.8.v20160314</version>
				<configuration>
					<webAppConfig>
						<!-- 指定 root context 在这里指定为${project.artifactId} 即 jetty， 那么访问时就用http://localized:8080/jetty 
							访问， 如果指定梶为test 就用http://localized:8080/test访问，更多信息，请查看jetty 插件官方文档 -->
						<contextPath>/${project.name}</contextPath>
						<defaultsDescriptor>src/test/resources/webdefault.xml</defaultsDescriptor>
					</webAppConfig>
					<stopPort>9966</stopPort>
					<stopKey>stop</stopKey>
					<stopWait>10</stopWait>
					<scanIntervalSeconds>10</scanIntervalSeconds>
					<!-- 指定额外需要监控变化的文件或文件夹，主要用于热部署中的识别文件更新 -->
					<scanTargetPatterns>
						<scanTargetPattern>
							<directory>src</directory>
							<includes>
								<!-- <include>*.java</include> <include>*.properties</include> -->
								<include>*.html</include>
								<include>*.js</include>
								<include>*.css</include>
							</includes>
							<!-- <excludes> <exclude>**/*.xml</exclude> <exclude>**/myspecial.properties</exclude> 
								</excludes> -->
						</scanTargetPattern>
					</scanTargetPatterns>
					<!-- 指定监控的扫描时间间隔，0为关闭jetty自身的热部署，主要是为了使用jrebel -->
					<!-- <scanIntervalSeconds>0</scanIntervalSeconds> -->
					<!-- 指定web页面的文件夹 -->
					<webAppSourceDirectory>${project.name}/src/main/webapp</webAppSourceDirectory>
				</configuration>
			</plugin>
		</plugins>
	
5.打jar包并加入jar包依赖文件

            <!-- 打包jar文件时，配置manifest文件，加入lib包的jar依赖 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-jar-plugin</artifactId>
				<configuration>
					<classesDirectory>target/classes/</classesDirectory>
					<archive>
						<manifest>
							<mainClass>com.test.container.TestMain</mainClass>
							<!-- 打包时 MANIFEST.MF文件不记录的时间戳版本 -->
							<useUniqueVersions>false</useUniqueVersions>
							<addClasspath>true</addClasspath>
							<classpathPrefix>lib/</classpathPrefix>
						</manifest>
						<manifestEntries>
							<Class-Path>.</Class-Path>
						</manifestEntries>
					</archive>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-dependency-plugin</artifactId>
				<executions>
					<execution>
						<id>copy-dependencies</id>
						<phase>package</phase>
						<goals>
							<goal>copy-dependencies</goal>
						</goals>
						<configuration>
							<type>jar</type>
							<includeTypes>jar</includeTypes>
							<useUniqueVersions>false</useUniqueVersions>
							<outputDirectory>
								${project.build.directory}/lib
							</outputDirectory>
						</configuration>
					</execution>
				</executions>
			</plugin>

6.编译install
 
              <plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<version>3.1</version>
					<configuration>
						<source>1.8</source>
						<target>1.8</target>
						<encoding>UTF-8</encoding>
					</configuration>
			 </plugin>
