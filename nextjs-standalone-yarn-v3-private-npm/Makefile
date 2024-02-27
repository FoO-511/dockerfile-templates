include .env.make

PROJECT_NAME = project_name
LOCAL_TAG = local
tag = dev
stage = production

test:
	docker run -p 3000:3000 ${REPOSITORY}:${tag}
login:
# change if you are not using aws ECR
	aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin ${REGISTRY}
build:
	docker build -t ${PROJECT_NAME}:${LOCAL_TAG} --build-arg NPM_REGISTRY=${NPM_REGISTRY} --build-arg NPM_REGISTRY_HTTP=${NPM_REGISTRY_HTTP} --build-arg NPM_TOKEN=${NPM_TOKEN} --build-arg STAGE=${stage} --build-arg NPM_SCOPE=${NPM_SCOPE} ./
tag: 
	docker tag ${PROJECT_NAME}:${LOCAL_TAG} ${REGISTRY}/${REPOSITORY}:${tag}
clean-dangling:
	docker image rm -f $$(docker images | grep -E '<none>[^\S\r\n]+<none>' | awk '{print $3}' | sort | uniq)
push: 
	docker push ${REGISTRY}/${REPOSITORY}:${tag}

publish: login build tag push