# 1. Transferring a robot
# 1.0 Initialization

```matlab:Code
clear;
```

# 1.1 The setup

```matlab:Code
% Given dynamics matrices 
A = [1  0   0.1   0;...
     0  1   0     0.1;...
     0  0   0.9   0;...
     0  0   0     0.9];
B = [0   0;...
     0   0;...
     0.1 0;...
     0   0.1];
n = 4;
m = 2; 

% Given parameters
T = 80;
p_initial = [0;5];
p_final = [15;-15];
w = [10 10; 20 10; 30 10; 30 0; 20 0; 10 -10]';
tau = [10 25 30 40 50 60];
U_max = 100;
E = [eye(2) zeros(2)];
```

# 1.2 The <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2^2"/> regularizer

```matlab:Code
controlSignalChanges = [];
meanDeviation = [];

for lambda_log = -3:3
    % ---------- Setup optimization problem ----------
    cvx_begin quiet
        % optimization variables
        variables x(n,T+1) u(m,T)
        % cost function
        minimize(sum(sum_square(E*x(:,tau+1)-w))+...
            (10^lambda_log)*sum(sum_square(u*(diag(-[ones(T-1,1);0])+diag(ones(T-1,1),-1)))));
        % subject to
        x(:,1) == [p_initial;zeros(2,1)];
        x(:,end) == [p_final;zeros(2,1)];
        for t = 1:T
            x(:,t+1) == A*x(:,t)+B*u(:,t);
            norm(u(:,t)) <= U_max;
        end
    cvx_end
    
    % ---------- Plot result ----------
    % trajectory
    figure('units','normalized','outerposition',[0 0 1 1]);
    hold on;
    set(gca,'FontSize',35);
    ax = gca;
    ax.XGrid = 'on';
    ax.YGrid = 'on';
    axis equal;
    title(strcat('$1.2\;a)\; \mathrm{Trajectory}\;\lambda =',sprintf('10^{%d}$',lambda_log)),'Interpreter','latex');
    scatter(x(1,:),x(2,:),70,'o','blue','LineWidth',3);
    scatter(x(1,tau+1),x(2,tau+1),300,'o','magenta','LineWidth',4);
    scatter(w(1,:),w(2,:),300,'s','red','LineWidth',4);
    legend('Trajectory','Waypoints (followed)', 'Waypoints (desired)','Location','best');
    ylabel('$p_y$','Interpreter','latex');
    xlabel('$p_x$','Interpreter','latex');
    saveas(gcf,sprintf('./transferring_a_robot_data/1_2_trajectory_1e%d.fig',lambda_log));
    saveas(gcf,sprintf('./transferring_a_robot_data/1_2_trajectory_1e%d.png',lambda_log));
    hold off; 
    
    % control signal
    figure('units','normalized','outerposition',[0 0 1 1]);
    hold on;
    set(gca,'FontSize',35);
    ax = gca;
    ax.XGrid = 'on';
    ax.YGrid = 'on';
    axis equal;
    title(strcat('$1.2\;b)\; \mathrm{Control}\;\mathrm{signal}\;\lambda =',sprintf('10^{%d}$',lambda_log)),'Interpreter','latex');
    scatter(0:T-1,u(1,:),70,'o','LineWidth',3);
    scatter(0:T-1,u(2,:),70,'o','LineWidth',3);
    legend({'$\mathbf{u}_1(t)$','$\mathbf{u}_2(t)$'},'Location','best','interpreter','latex');
    ylabel('$ $','Interpreter','latex');
    xlabel('$t$','Interpreter','latex');
    saveas(gcf,sprintf('./transferring_a_robot_data/1_2_controlSignal_1e%d.fig',lambda_log));
    saveas(gcf,sprintf('./transferring_a_robot_data/1_2_controlSignal_1e%d.png',lambda_log));
    hold off;
    
    controlSignalChangesCur = 0;
    for t = 1:T-1
        if norm(u(:,t+1)-u(:,t))>1e-4
            controlSignalChangesCur = controlSignalChangesCur+1;
        end
    end
    controlSignalChanges = [controlSignalChanges [10^lambda_log;controlSignalChangesCur]]; 
    fprintf("1.2 c) For lambda = 1e%d there were %d optimal control signal changes.\n",lambda_log,controlSignalChangesCur);
    
    meanDeviationCur = (1/length(w))*sum(sqrt(sum_square(E*x(:,tau+1)-w)));
    meanDeviation = [meanDeviation [10^lambda_log; meanDeviationCur]];
    fprintf("1.2 d) For lambda = 1e%d the mean deviation from the waypoints is %f.\n",lambda_log,meanDeviationCur);
end
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_0.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_0.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_1.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_1.png
)


```text:Output
1.2 c) For lambda = 1e-3 there were 79 optimal control signal changes.
1.2 d) For lambda = 1e-3 the mean deviation from the waypoints is 0.125677.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_2.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_2.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_3.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_3.png
)


```text:Output
1.2 c) For lambda = 1e-2 there were 79 optimal control signal changes.
1.2 d) For lambda = 1e-2 the mean deviation from the waypoints is 0.824170.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_4.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_4.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_5.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_5.png
)


```text:Output
1.2 c) For lambda = 1e-1 there were 79 optimal control signal changes.
1.2 d) For lambda = 1e-1 the mean deviation from the waypoints is 2.195805.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_6.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_6.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_7.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_7.png
)


```text:Output
1.2 c) For lambda = 1e0 there were 79 optimal control signal changes.
1.2 d) For lambda = 1e0 the mean deviation from the waypoints is 3.682588.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_8.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_8.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_9.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_9.png
)


```text:Output
1.2 c) For lambda = 1e1 there were 79 optimal control signal changes.
1.2 d) For lambda = 1e1 the mean deviation from the waypoints is 5.631691.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_10.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_10.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_11.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_11.png
)


```text:Output
1.2 c) For lambda = 1e2 there were 79 optimal control signal changes.
1.2 d) For lambda = 1e2 the mean deviation from the waypoints is 10.904150.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_12.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_12.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_13.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_13.png
)


```text:Output
1.2 c) For lambda = 1e3 there were 79 optimal control signal changes.
1.2 d) For lambda = 1e3 the mean deviation from the waypoints is 15.330435.
```


```matlab:Code
save('./transferring_a_robot_data/1_2_data_c_d.mat','controlSignalChanges','meanDeviation');

```

# 1.3 The <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2"/> regularizer

```matlab:Code
controlSignalChanges = [];
controlSignalChangesL1 = [];
meanDeviation = [];

for lambda_log = -3:3
    % ---------- Setup optimization problem ----------
    cvx_begin quiet
        % optimization variables
        variables x(n,T+1) u(m,T)
        % cost function
        minimize(sum(sum_square(E*x(:,tau+1)-w))+...
            (10^lambda_log)*sum(norms(u*(diag(-[ones(T-1,1);0])+diag(ones(T-1,1),-1)),2)));
        % subject to
        x(:,1) == [p_initial;zeros(2,1)];
        x(:,end) == [p_final;zeros(2,1)];
        for t = 1:T
            x(:,t+1) == A*x(:,t)+B*u(:,t);
            norm(u(:,t)) <= U_max;
        end
    cvx_end
    
    % ---------- Plot result ----------
    % trajectory
    figure('units','normalized','outerposition',[0 0 1 1]);
    hold on;
    set(gca,'FontSize',35);
    ax = gca;
    ax.XGrid = 'on';
    ax.YGrid = 'on';
    axis equal;
    title(strcat('$1.3\;a)\; \mathrm{Trajectory}\;\lambda =',sprintf('10^{%d}$',lambda_log)),'Interpreter','latex');
    scatter(x(1,:),x(2,:),70,'o','blue','LineWidth',3);
    scatter(x(1,tau+1),x(2,tau+1),300,'o','magenta','LineWidth',4);
    scatter(w(1,:),w(2,:),300,'s','red','LineWidth',4);
    legend('Trajectory','Waypoints (followed)', 'Waypoints (desired)','Location','best');
    ylabel('$p_y$','Interpreter','latex');
    xlabel('$p_x$','Interpreter','latex');
    saveas(gcf,sprintf('./transferring_a_robot_data/1_3_trajectory_1e%d.fig',lambda_log));
    saveas(gcf,sprintf('./transferring_a_robot_data/1_3_trajectory_1e%d.png',lambda_log));
    hold off; 
    
    % control signal
    figure('units','normalized','outerposition',[0 0 1 1]);
    hold on;
    set(gca,'FontSize',35);
    ax = gca;
    ax.XGrid = 'on';
    ax.YGrid = 'on';
    axis equal;
    title(strcat('$1.3\;b)\; \mathrm{Control}\;\mathrm{signal}\;\lambda =',sprintf('10^{%d}$',lambda_log)),'Interpreter','latex');
    scatter(0:T-1,u(1,:),70,'o','LineWidth',3);
    scatter(0:T-1,u(2,:),70,'o','LineWidth',3);
    legend({'$\mathbf{u}_1(t)$','$\mathbf{u}_2(t)$'},'Location','best','interpreter','latex');
    ylabel('$ $','Interpreter','latex');
    xlabel('$t$','Interpreter','latex');
    saveas(gcf,sprintf('./transferring_a_robot_data/1_3_controlSignal_1e%d.fig',lambda_log));
    saveas(gcf,sprintf('./transferring_a_robot_data/1_3_controlSignal_1e%d.png',lambda_log));
    hold off;
    
    controlSignalChangesCur = 0;
    for t = 1:T-1
        if norm(u(:,t+1)-u(:,t))>1e-4
            controlSignalChangesCur = controlSignalChangesCur+1;
        end
    end
    controlSignalChanges = [controlSignalChanges [10^lambda_log;controlSignalChangesCur]]; 
    fprintf("1.3 c) For lambda = 1e%d there were %d optimal control signal changes.\n",lambda_log,controlSignalChangesCur);
    
    controlSignalChangesCurL1 = 0;
    for t = 1:T-1
        if norms(u(:,t+1)-u(:,t),1)>1e-4
            controlSignalChangesCurL1 = controlSignalChangesCurL1+1;
        end
    end
    controlSignalChangesL1 = [controlSignalChangesL1 [10^lambda_log;controlSignalChangesCurL1]]; 
    fprintf("1.3 c) For lambda = 1e%d there were %d optimal control signal L1 changes.\n",lambda_log,controlSignalChangesCurL1);
    
    meanDeviationCur = (1/length(w))*sum(sqrt(sum_square(E*x(:,tau+1)-w)));
    meanDeviation = [meanDeviation [10^lambda_log; meanDeviationCur]];
    fprintf("1.3 d) For lambda = 1e%d the mean deviation from the waypoints is %f.\n",lambda_log,meanDeviationCur);
end
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_14.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_14.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_15.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_15.png
)


```text:Output
1.3 c) For lambda = 1e-3 there were 7 optimal control signal changes.
1.3 c) For lambda = 1e-3 there were 7 optimal control signal L1 changes.
1.3 d) For lambda = 1e-3 the mean deviation from the waypoints is 0.007501.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_16.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_16.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_17.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_17.png
)


```text:Output
1.3 c) For lambda = 1e-2 there were 7 optimal control signal changes.
1.3 c) For lambda = 1e-2 there were 7 optimal control signal L1 changes.
1.3 d) For lambda = 1e-2 the mean deviation from the waypoints is 0.074656.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_18.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_18.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_19.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_19.png
)


```text:Output
1.3 c) For lambda = 1e-1 there were 8 optimal control signal changes.
1.3 c) For lambda = 1e-1 there were 8 optimal control signal L1 changes.
1.3 d) For lambda = 1e-1 the mean deviation from the waypoints is 0.702073.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_20.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_20.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_21.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_21.png
)


```text:Output
1.3 c) For lambda = 1e0 there were 4 optimal control signal changes.
1.3 c) For lambda = 1e0 there were 4 optimal control signal L1 changes.
1.3 d) For lambda = 1e0 the mean deviation from the waypoints is 2.887570.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_22.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_22.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_23.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_23.png
)


```text:Output
1.3 c) For lambda = 1e1 there were 3 optimal control signal changes.
1.3 c) For lambda = 1e1 there were 3 optimal control signal L1 changes.
1.3 d) For lambda = 1e1 the mean deviation from the waypoints is 5.368939.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_24.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_24.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_25.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_25.png
)


```text:Output
1.3 c) For lambda = 1e2 there were 2 optimal control signal changes.
1.3 c) For lambda = 1e2 there were 2 optimal control signal L1 changes.
1.3 d) For lambda = 1e2 the mean deviation from the waypoints is 12.591431.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_26.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_26.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_27.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_27.png
)


```text:Output
1.3 c) For lambda = 1e3 there were 1 optimal control signal changes.
1.3 c) For lambda = 1e3 there were 1 optimal control signal L1 changes.
1.3 d) For lambda = 1e3 the mean deviation from the waypoints is 16.226625.
```


```matlab:Code
save('./transferring_a_robot_data/1_3_data_c_d.mat','controlSignalChanges','controlSignalChangesL1','meanDeviation');

```

# 1.4 The <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_1"/> regularizer

```matlab:Code
controlSignalChanges = [];
controlSignalChangesL1 = [];
meanDeviation = [];

for lambda_log = -3:3
    % ---------- Setup optimization problem ----------
    cvx_begin quiet
        % optimization variables
        variables x(n,T+1) u(m,T)
        % cost function
        minimize(sum(sum_square(E*x(:,tau+1)-w))+...
            (10^lambda_log)*sum(norms(u*(diag(-[ones(T-1,1);0])+diag(ones(T-1,1),-1)),1)));
        % subject to
        x(:,1) == [p_initial;zeros(2,1)];
        x(:,end) == [p_final;zeros(2,1)];
        for t = 1:T
            x(:,t+1) == A*x(:,t)+B*u(:,t);
            norm(u(:,t)) <= U_max;
        end
    cvx_end
    
    % ---------- Plot result ----------
    % trajectory
    figure('units','normalized','outerposition',[0 0 1 1]);
    hold on;
    set(gca,'FontSize',35);
    ax = gca;
    ax.XGrid = 'on';
    ax.YGrid = 'on';
    axis equal;
    title(strcat('$1.4\;a)\; \mathrm{Trajectory}\;\lambda =',sprintf('10^{%d}$',lambda_log)),'Interpreter','latex');
    scatter(x(1,:),x(2,:),70,'o','blue','LineWidth',3);
    scatter(x(1,tau+1),x(2,tau+1),300,'o','magenta','LineWidth',4);
    scatter(w(1,:),w(2,:),300,'s','red','LineWidth',4);
    legend('Trajectory','Waypoints (followed)', 'Waypoints (desired)','Location','best');
    ylabel('$p_y$','Interpreter','latex');
    xlabel('$p_x$','Interpreter','latex');
    saveas(gcf,sprintf('./transferring_a_robot_data/1_4_trajectory_1e%d.fig',lambda_log));
    saveas(gcf,sprintf('./transferring_a_robot_data/1_4_trajectory_1e%d.png',lambda_log));
    hold off; 
    
    % control signal
    figure('units','normalized','outerposition',[0 0 1 1]);
    hold on;
    set(gca,'FontSize',35);
    ax = gca;
    ax.XGrid = 'on';
    ax.YGrid = 'on';
    axis equal;
    title(strcat('$1.4\;b)\; \mathrm{Control}\;\mathrm{signal}\;\lambda =',sprintf('10^{%d}$',lambda_log)),'Interpreter','latex');
    scatter(0:T-1,u(1,:),70,'o','LineWidth',3);
    scatter(0:T-1,u(2,:),70,'o','LineWidth',3);
    legend({'$\mathbf{u}_1(t)$','$\mathbf{u}_2(t)$'},'Location','best','interpreter','latex');
    ylabel('$ $','Interpreter','latex');
    xlabel('$t$','Interpreter','latex');
    saveas(gcf,sprintf('./transferring_a_robot_data/1_4_controlSignal_1e%d.fig',lambda_log));
    saveas(gcf,sprintf('./transferring_a_robot_data/1_4_controlSignal_1e%d.png',lambda_log));
    hold off;
    
    controlSignalChangesCur = 0;
    for t = 1:T-1
        if norm(u(:,t+1)-u(:,t))>1e-4
            controlSignalChangesCur = controlSignalChangesCur+1;
        end
    end
    controlSignalChanges = [controlSignalChanges [10^lambda_log;controlSignalChangesCur]]; 
    fprintf("1.4 c) For lambda = 1e%d there were %d optimal control signal changes.\n",lambda_log,controlSignalChangesCur);
    
    controlSignalChangesCurL1 = 0;
    for t = 1:T-1
        if norms(u(:,t+1)-u(:,t),1)>1e-4
            controlSignalChangesCurL1 = controlSignalChangesCurL1+1;
        end
    end
    controlSignalChangesL1 = [controlSignalChangesL1 [10^lambda_log;controlSignalChangesCurL1]]; 
    fprintf("1.4 c) For lambda = 1e%d there were %d optimal control signal L1 changes.\n",lambda_log,controlSignalChangesCurL1);    
    
    meanDeviationCur = (1/length(w))*sum(sqrt(sum_square(E*x(:,tau+1)-w)));
    meanDeviation = [meanDeviation [10^lambda_log; meanDeviationCur]];
    fprintf("1.4 d) For lambda = 1e%d the mean deviation from the waypoints is %f.\n",lambda_log,meanDeviationCur);
    
end
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_28.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_28.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_29.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_29.png
)


```text:Output
1.4 c) For lambda = 1e-3 there were 11 optimal control signal changes.
1.4 c) For lambda = 1e-3 there were 11 optimal control signal L1 changes.
1.4 d) For lambda = 1e-3 the mean deviation from the waypoints is 0.010735.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_30.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_30.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_31.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_31.png
)


```text:Output
1.4 c) For lambda = 1e-2 there were 11 optimal control signal changes.
1.4 c) For lambda = 1e-2 there were 11 optimal control signal L1 changes.
1.4 d) For lambda = 1e-2 the mean deviation from the waypoints is 0.105450.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_32.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_32.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_33.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_33.png
)


```text:Output
1.4 c) For lambda = 1e-1 there were 14 optimal control signal changes.
1.4 c) For lambda = 1e-1 there were 14 optimal control signal L1 changes.
1.4 d) For lambda = 1e-1 the mean deviation from the waypoints is 0.886316.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_34.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_34.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_35.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_35.png
)


```text:Output
1.4 c) For lambda = 1e0 there were 9 optimal control signal changes.
1.4 c) For lambda = 1e0 there were 9 optimal control signal L1 changes.
1.4 d) For lambda = 1e0 the mean deviation from the waypoints is 2.873231.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_36.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_36.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_37.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_37.png
)


```text:Output
1.4 c) For lambda = 1e1 there were 4 optimal control signal changes.
1.4 c) For lambda = 1e1 there were 4 optimal control signal L1 changes.
1.4 d) For lambda = 1e1 the mean deviation from the waypoints is 5.436149.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_38.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_38.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_39.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_39.png
)


```text:Output
1.4 c) For lambda = 1e2 there were 2 optimal control signal changes.
1.4 c) For lambda = 1e2 there were 2 optimal control signal L1 changes.
1.4 d) For lambda = 1e2 the mean deviation from the waypoints is 13.027273.
```


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_40.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_40.png
)


![/Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_41.png
](transferring_a_robot_sol_images//Users/leonardopedroso/Desktop/OA/Project/part1/transferring_a_robot_sol_images/figure_41.png
)


```text:Output
1.4 c) For lambda = 1e3 there were 2 optimal control signal changes.
1.4 c) For lambda = 1e3 there were 2 optimal control signal L1 changes.
1.4 d) For lambda = 1e3 the mean deviation from the waypoints is 16.046286.
```


```matlab:Code
save('./transferring_a_robot_data/1_4_data_c_d.mat','controlSignalChanges','meanDeviation','controlSignalChangesL1');
```

# Comments


"Task 4. Comment on what you have observed from Tasks 1 to 3 (for example, compare the impact of the three regularizers on the optimal control signal that they each induce)."


  


Mean deviation from waypoints



<img src="https://latex.codecogs.com/gif.latex?\lambda"/>       <img src="https://latex.codecogs.com/gif.latex?10^{-3}"/>        <img src="https://latex.codecogs.com/gif.latex?10^{-2}"/>       <img src="https://latex.codecogs.com/gif.latex?10^{-1}"/>       <img src="https://latex.codecogs.com/gif.latex?10^{0}"/>         <img src="https://latex.codecogs.com/gif.latex?10^{1}"/>        <img src="https://latex.codecogs.com/gif.latex?10^{2}"/>          <img src="https://latex.codecogs.com/gif.latex?10^{3}"/>



<img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2^2"/>       0.1257    0.8242    2.1958    3.6826    5.6317   10.9041   15.3304




<img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2"/>       0.0075    0.0747    0.7021    2.8876    5.3689   12.5914   16.2266




<img src="https://latex.codecogs.com/gif.latex?\inline&space;l_1"/>       0.0107    0.1055    0.8863    2.8732    5.4361   13.0273   16.0463


  


Number of changes (L2) in the optimal control signal



<img src="https://latex.codecogs.com/gif.latex?\lambda"/>       <img src="https://latex.codecogs.com/gif.latex?10^{-3}"/>        <img src="https://latex.codecogs.com/gif.latex?10^{-2}"/>       <img src="https://latex.codecogs.com/gif.latex?10^{-1}"/>       <img src="https://latex.codecogs.com/gif.latex?10^{0}"/>         <img src="https://latex.codecogs.com/gif.latex?10^{1}"/>        <img src="https://latex.codecogs.com/gif.latex?10^{2}"/>          <img src="https://latex.codecogs.com/gif.latex?10^{3}"/>



<img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2^2"/>       79            79           79          79           79          79            79




<img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2"/>      10             8             10          5             4            3              1




<img src="https://latex.codecogs.com/gif.latex?\inline&space;l_1"/>      13             12           14          9             6            3              3


  


First, it is possible to notice that using the <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2^2"/> regularizer, whatever the value of <img src="https://latex.codecogs.com/gif.latex?\inline&space;\lambda"/> is there is always the maximum number of changes of the optimal control signal. Using this regularizer, the optimal control obtained is a smooth continuous signal. In fact, the weighing factor is the square of the difference. Once the difference is low (<<1) the square reduces this difference even more. This way, this regularizer penalizes very significantly big differences, but is benevolent when the diffferences are small. This behaviour is not ideal for problems where there may be outliers, which is not the case. This originates a smooth signal.




Second, both the <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2"/> and <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_1"/> regularizers do not penalize big differences significantly and are not as benevolent with small differences, as it is the case with the <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2^2"/> regularizer. In fact, this results, for both aproaches in a roughly piecewise constant signal for every  value of <img src="https://latex.codecogs.com/gif.latex?\inline&space;\lambda"/>. Given that the <img src="https://latex.codecogs.com/gif.latex?\inline&space;||x||_1&space;\geq&space;||x||_2"/>, then, for the same value of <img src="https://latex.codecogs.com/gif.latex?\inline&space;\lambda"/> it is expected that the <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_1"/> regularizer penallizes changes in the optimal control signal more than the <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2"/> regularizer. As expected, given that more importance is given to the regularizer, the performance as far as waypoint tracking is concerned decreases slightly for the <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_1"/> regularizer when compared with the <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_2"/> regularizer. Furthermore, it is also notiable that the <img src="https://latex.codecogs.com/gif.latex?\inline&space;l_1"/> norm, also because of the aforementioned increse of regularizer importance and the fact that it is the sum of the absloute values of the components of a vector, is more prone to originate sparse control signals.

