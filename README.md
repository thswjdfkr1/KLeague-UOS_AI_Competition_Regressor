# KLeague-UOS_Public_AI_Competition  
K리그 경기 내 주어진 플레이 시퀀스의 마지막 패스 도착 좌표(X,Y)를 예측하는 AI 모델을 개발    
 
# 주제  
K리그 경기 내 최종 패스 좌표 예측 AI 모델 개발    

# 평가지표   
유클리드 거리(Euclidean Distance)           

# 사용기술
- Python
- LSTM
- TabPFN
- K-Fold

# 데이터 전처리   
1. 데이터 전처리
   - y축 상하 반전을 통한 데이터 증강
   - 특성별 파생변수 생성
2. 범주형 변수 정규화
3. 데이터셋 구축
   - 수치형, 범주형 변수 중 시계열 특성을 가진 변수 평탄화(마지막 시점 이전 3개의 데이터)
   - 'game_episode'별 그룹화로 최종 데이터셋 구축

# 추론
1. TabPFNRegressor 모델 구축
2. 2-Stage Residual Learning(잔차 학습)
   - 'player_id_curr_t0', 'team_id_curr_t0', 'new_opp_team_id_curr_t0' 기준으로 outer fold 내 평균 dx, dy 기반 Target Encoding 생성 (Outer Fold 기준으로 계산하여 데이터 누수 방지)
   -'game_id' 기준 5-Fold로 outer_GroupKFold 생성
   - 각 outer_GroupKFold 내부에서 3-Fold로 inner_GroupKFold 기반으로 stage1 OOF 예측 생성(Residual 안정적 학습 목적)
   - stage1
     * dx, dy 직접 예측
   - stage2
     * stage1 OOF 기반 Residual(dx, dy) 학습 및 예측
     * 최종 이동거리(final_dx, final_dy) = stage1 + stage2
3. 최종 좌표 복원
   -원 start_x + final_dx
   - 원 start_y + final_dy
4. Fold별 test 예측 평균 앙상블 통한 최종 좌표 추론
