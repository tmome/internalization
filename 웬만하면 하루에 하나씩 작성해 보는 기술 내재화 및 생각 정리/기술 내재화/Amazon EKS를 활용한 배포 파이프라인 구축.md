# 까먹지 않으려고 작성하는 내가 이해한 Amazon EKS와 Argo CD를 활용한 EC2 기반 CI/CD 파이프라인 구축
>
> ## CI/CD 파이프라인 흐름
>
> 1. **개발자가 새로운 기능이나 수정 사항을 GitHub 저장소에 커밋**
  >    - 개발자가 코드를 GitHub 저장소에 커밋하여 변경 사항을 반영
>
> 2. **GitHub Actions을 통해 파이프라인을 트리거**
  >    - 커밋이 발생하면 GitHub Actions 등의 CI 도구가 파이프라인을 트리거한다.
>
> 3. **CI 파이프라인에서 Docker 이미지를 빌드하고 AWS ECR에 푸시**
  >    - CI 파이프라인에서 Docker 이미지를 빌드한 후, 이를 Amazon ECR (Elastic Container Registry)에 푸시
>
> 4. **Kubernetes Manifest 업데이트 및 커밋**
  >    - 새로운 Docker 이미지 태그를 포함한 Kubernetes manifest 파일을 업데이트합니다. 이 업데이트는 GitHub 저장소에 커밋
> 5. **Argo CD는 Git 리포지토리에서 Manifest를 모니터링**
  >    - Argo CD는 GitHub 저장소에서 Kubernetes manifest 파일의 변화를 감지하고, 변경 사항을 자동으로 EKS 클러스터에 배포
>
> - 참고 이미지
> 
> <img width="1873" alt="스크린샷 2024-07-20 오후 4 23 16" src="https://github.com/user-attachments/assets/a09e2523-4324-486b-93dd-635bb47929ad">

## Amazon EKS, EC2, ECR 주요 구성 요소

> | 항목 | Amazon EKS | Amazon EC2 | Amazon ECR |
> |---|---|---|---|
> | **설명** | Kubernetes를 관리형으로 제공하는 서비스 | AWS의 가상 서버 서비스 | Docker 이미지 저장 및 관리 서비스 |
> | **주요 기능** | - Kubernetes 클러스터를 통해 컨테이너 오케스트레이션 <br> - EKS 클러스터, 노드 그룹, 파드 | - 가상 서버 프로비저닝 <br> - 자동 스케일링 <br> - 네트워킹 구성 | - Docker 이미지 저장소 <br> - 이미지 버전 관리 <br> - CI/CD 통합 |
> | **구성 요소** | - EKS 클러스터 <br> - 노드 그룹 <br> - 파드 | - EC2 인스턴스 | - ECR 레포지토리 |
> | **사용 사례** | - Kubernetes 생태계 활용 <br> - 멀티 클라우드 또는 온프레미스 환경 | - 웹 서버 <br> - 데이터베이스 <br> - 애플리케이션 서버 | - 컨테이너 이미지 배포 <br> - 이미지 관리와 저장 |


