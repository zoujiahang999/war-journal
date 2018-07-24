# war-journal
#include "gamewidget1.h"
#include <QDebug>

Gamewidget1::Gamewidget1(QWidget *parent) : QWidget(parent)
{
    this->resize(800,1080);

    score=0;
    boom_counter=5;
    pause=false;

    block_time=1;
    block_interval=0;
    send_time=0;
    send_interval=0;
    rwd_bullet_stnTime=0;

    player=FlyFactory::GenPlane("normal");
    this->setCursor(Qt::BlankCursor);
    this->cursor().setPos(QPoint(player.X+this->x(),player.Y+this->y()));
    connect(&t_out,SIGNAL(timeout()),this,SLOT(TimeDraw()));
    t_out.setInterval(20);
    send_interval=100/t_out.interval();
    block_interval=260/t_out.interval();
    t_out.start();

    GameOver=false;

    player.X=this->width()/2-player.Width/2;
    player.Y=this->height()-player.Height/2;

    music_explosion=new QSoundEffect(this);
}

Plane FlyFactory::GenPlane(QString style){
    if(style=="normal") {
        QImage tempBmp=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/playerplane.png");
        return Plane("normal",600,500,0,0,tempBmp.width(),tempBmp.height(),tempBmp);
    }
    else if(style=="super"){
        QImage tempBmp=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/playerplane.png");
        return  Plane("super",600,500,0,0,tempBmp.width(),tempBmp.height(),tempBmp);
    }
}

Bullet FlyFactory::GenBullet(QString style, int p_x, int p_y,int speed_x,int speed_y){
    if(style=="red"){
        QImage tempBmp=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/playerbullet.png");
        return Bullet("red",p_x,p_y,0,20,tempBmp.width(),tempBmp.height(),tempBmp);
    }
    else if(style=="blue"){
        QImage tempBmp=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/playerbullet.png");
        return Bullet("blue",p_x,p_y,speed_x,speed_y,tempBmp.width(),tempBmp.height(),tempBmp);
    }
}

Enemy FlyFactory::GenEnemy(QString size, int speedBase, int wid){
    if(size=="big"){
        QImage tempBmp=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemyplane3.png");
        return Enemy("big",0,400,6,qrand()%2-1+speedBase,tempBmp.width(),tempBmp.height(),1600,tempBmp);
    }
    else if(size=="mid"){
        QImage tempBmp=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemyplane2.png");
        return Enemy("mid",qrand()%wid,0,qAbs(qrand()%7+1),qrand()%9+1+speedBase,tempBmp.width(),tempBmp.height(),800,tempBmp);
    }
    else if(size=="small"){
        QImage tempBmp=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemyplane.png");
        return Enemy("small",qrand()%wid,0,qAbs(qrand()%5+1),qrand()%4+1+speedBase,tempBmp.width(),tempBmp.height(),600,tempBmp);
    }
    else if(size=="boss"){
        QImage tempBmp=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/boss1.png");
        return Enemy("boss",qrand()%wid,0,qAbs(qrand()%3+1),qrand()%3+1+speedBase,tempBmp.width(),tempBmp.height(),4000,tempBmp);
    }
}

Explosion FlyFactory::GenExplosion(QString style, int p_x, int p_y){
    QImage tempBmp1=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemy1dead1.png");
    QImage tempBmp2=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemy2dead1.png");
    QImage tempBmp3=QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemy3dead1.png");
    if(style=="small"){
        return Explosion("small",p_x,p_y,300,tempBmp1);
    }
    else if(style=="mid"){
        return Explosion("mid",p_x,p_y,300,tempBmp2);
    }
    else if(style=="big"){
        return Explosion("big",p_x,p_y,300,tempBmp3);
    }
}

void Gamewidget1::TimeDraw(){
    t_out.stop();

    if(player.level==1&&score==0&&start_number==0){
        start_number++;

        t_out.stop();
        pause=true;
        setMouseTracking(true);
        this->setCursor(Qt::OpenHandCursor);

        start=new QLabel(this);
        start->resize(this->width(),this->height());
        start->move(0,0);
        start->setPixmap(QPixmap("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/diary.png"));

        QIcon nextbutton("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/nextbutton.png");

        next=new QPushButton(start);

        next->resize(200,75);
        next->setIcon(nextbutton);
        next->setIconSize(QSize(200,75));
        next->setFlat(true);
        next->move(this->width()-380,this->height()-180);


        start->show();

        connect(next,SIGNAL(clicked(bool)),this,SLOT(G_next()));
        connect(next,SIGNAL(clicked(bool)),start,SLOT(close()));
    }
    if(player.level==1&&score>=10000&&enemy_list.size()-1==0){
        G_succeed();
    }
    else if(player.level==2&&score>=10000&&enemy_list.size()-1==0){
        G_succeed();
    }

    //关卡

    if(pause){
        update();
        t_out.start(10);
        return;
    }//暂停

    if(send_time>send_interval){
        if(rwd_bullet_stnTime){
            player_Bullet_list.push_back(FlyFactory::GenBullet("blue",player.X-6,player.Y-20,-5,10));
            player_Bullet_list.push_back(FlyFactory::GenBullet("blue",player.X,player.Y-20,0,10));
            player_Bullet_list.push_back(FlyFactory::GenBullet("blue",player.X+6,player.Y-20,5,10));
        }
        else{
            player_Bullet_list.push_back((FlyFactory::GenBullet("red",player.X,player.Y-20,0,10)));
        }
        send_time=0;
    }//玩家射子弹

    if((t=qrand()%4)==0){
        for(int i=1;i<enemy_list.size()-1;i++){
            if(enemy_list[i].Name=="boss") {
                if(t%80==0)
                    enemy_list[i].Bullet_list.push_back(FlyFactory::GenBullet("red",enemy_list[i].X,enemy_list[i].Y+enemy_list[i].Height/2,0,10));
                else if(t%120==0)
                    enemy_list[i].Bullet_list.push_back(FlyFactory::GenBullet("blue",enemy_list[i].X-6,enemy_list[i].Y-20,-5,10));
                    enemy_list[i].Bullet_list.push_back(FlyFactory::GenBullet("blue",enemy_list[i].X-6,enemy_list[i].Y-20,-2.5,10));
                    enemy_list[i].Bullet_list.push_back(FlyFactory::GenBullet("blue",enemy_list[i].X-6,enemy_list[i].Y-20,2.5,10));
                    enemy_list[i].Bullet_list.push_back(FlyFactory::GenBullet("blue",enemy_list[i].X-6,enemy_list[i].Y-20,5,10));
            }
            else if(enemy_list[i].Name=="small")
                if(t%40==0)
                    enemy_list[i].Bullet_list.push_back(FlyFactory::GenBullet("red",enemy_list[i].X,enemy_list[i].Y+enemy_list[i].Height/2,0,10));
            else if(enemy_list[i].Name=="mid")
                if(t%40==0)
                    enemy_list[i].Bullet_list.push_back(FlyFactory::GenBullet("red",enemy_list[i].X,enemy_list[i].Y+enemy_list[i].Height/2,0,10));        }
    }//敌人射子弹

    if(block_time%block_interval==0){
        if(player.level==1&&score<=10000){
            int speedBase=1;
            if(block_interval<2) speedBase=2;
            else if(block_interval>=2&&block_interval<5) speedBase=3;
            else if(block_interval>=5&&block_interval<10) speedBase=2;
            if(block_time%(block_interval*3)==0&&boss_set!=2) enemy_list.push_back(FlyFactory::GenEnemy("small",speedBase,this->width()));
        }
        else if(player.level==2&&score<=10000){
            int speedBase=2;
            if(block_interval<2) speedBase=2;
            else if(block_interval>=2&&block_interval<5) speedBase=3;
            else if(block_interval>=5&&block_interval<10) speedBase=2;
            if(block_time%(block_interval*4)==0) enemy_list.push_back(FlyFactory::GenEnemy("small",speedBase,this->width()));
            else if(block_time%(block_interval*5)==0) enemy_list.push_back(FlyFactory::GenEnemy("mid",speedBase,this->width()));
        }
        else if(player.level==3){
            int speedBase=3;
            if(block_interval<2) speedBase=2;
            else if(block_interval>=2&&block_interval<5) speedBase=3;
            else if(block_interval>=5&&block_interval<10) speedBase=2;
            if(score>10000&&boss_set!=2) {
                enemy_list.push_back(FlyFactory::GenEnemy("boss",1,this->width()));
                boss_set++;
            }
            else if(block_time%(block_interval*8)==0&&boss_set!=2) enemy_list.push_back(FlyFactory::GenEnemy("small",speedBase,this->width()));
            else if(block_time%(block_interval*10)==0&&boss_set!=2) enemy_list.push_back(FlyFactory::GenEnemy("mid",speedBase,this->width()));
            else if(block_time%(block_interval*6)==0&&boss_set!=2) enemy_list.push_back(FlyFactory::GenEnemy("big",speedBase,this->width()));
        }
    }//生成敌人

    send_time++;
    block_time++;

    for(int i=0;i<enemy_list.size()-1;i++)
        for(int j=0;j<player_Bullet_list.size()-1;j++){
                if(qAbs(player_Bullet_list[j].X-enemy_list[i].X)<((player_Bullet_list[j].Width+enemy_list[i].Width)/2-5)&&qAbs(player_Bullet_list[j].Y-enemy_list[i].Y)<((player_Bullet_list[j].Height+enemy_list[i].Height)/2-5)){
                    if(player_Bullet_list[j].Name=="red"){
                        enemy_list[i].HP-=200;
                        if(enemy_list[i].HP<=0){
                            if(enemy_list[i].Name!="boss") {
                                score+=1000;
                                if(enemy_list[i].Name=="small") {
                                    explosion_list.push_back((FlyFactory::GenExplosion("small",enemy_list[i].X,enemy_list[i].Y)));
                                    explosion_list2.push_back((FlyFactory::GenExplosion("small",enemy_list[i].X,enemy_list[i].Y)));
                                }
                                else if(enemy_list[i].Name=="mid"){
                                    explosion_list.push_back((FlyFactory::GenExplosion("mid",enemy_list[i].X,enemy_list[i].Y)));
                                    explosion_list2.push_back((FlyFactory::GenExplosion("mid",enemy_list[i].X,enemy_list[i].Y)));
                                }
                                else if(enemy_list[i].Name=="big"){
                                    explosion_list.push_back((FlyFactory::GenExplosion("big",enemy_list[i].X,enemy_list[i].Y)));
                                    explosion_list2.push_back((FlyFactory::GenExplosion("big",enemy_list[i].X,enemy_list[i].Y)));
                                }
                                music_explosion->setSource(QUrl::fromLocalFile("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/sound/freeze.wav"));
                                music_explosion->setLoopCount(1);
                                music_explosion->setVolume(0.5);
                                music_explosion->play();
                            }
                            else if(enemy_list[i].Name=="boss") {
                                b_x=enemy_list[i].X;
                                b_y=enemy_list[i].Y;
                                b_wid=enemy_list[i].Width;
                                b_heig=enemy_list[i].Height;
                                GG=1;
                            }
                            enemy_list.removeAt(i);
                        }
                    }
                    else if(player_Bullet_list[j].Name=="blue"){
                        enemy_list[i].HP-=100;
                        if(enemy_list[i].HP<=0){
                            if(enemy_list[i].Name!="boss") {
                                score+=1000;
                                if(enemy_list[i].Name=="small") explosion_list.push_back((FlyFactory::GenExplosion("small",enemy_list[i].X,enemy_list[i].Y)));
                                else if(enemy_list[i].Name=="mid") explosion_list.push_back((FlyFactory::GenExplosion("mid",enemy_list[i].X,enemy_list[i].Y)));
                                else if(enemy_list[i].Name=="big") explosion_list.push_back((FlyFactory::GenExplosion("big",enemy_list[i].X,enemy_list[i].Y)));
                            }
                            else if(enemy_list[i].Name=="boss") G_succeed();
                            enemy_list.removeAt(i);
                        }
                    }
                    player_Bullet_list.removeAt(j);
        }
    }//敌人与自己子弹碰撞

    for(int i=0;i<enemy_list.size()-1;i++)
        for(int j=0;j<enemy_list[i].Bullet_list.size()-1;j++){
            if(qAbs(player.X-enemy_list[i].Bullet_list[j].X)<((50+enemy_list[i].Bullet_list[j].Width)/2-5)&&qAbs(player.Y-enemy_list[i].Bullet_list[j].Y)<((70+enemy_list[i].Bullet_list[j].Height)/2-5)){
                player.HP--;
                enemy_list[i].Bullet_list.removeAt(j);
                if(player.HP<=0){
                    t_out.stop();
                    this->setCursor(Qt::ForbiddenCursor);
                    GameOver=true;
                    update();
                    return;
                }
            }
    }//敌人子弹与自己碰撞

    for(int i=0;i<enemy_list.size()-1;i++){
        if(qAbs(player.X-enemy_list[i].X)<((50+enemy_list[i].Width)/2-5)&&qAbs(player.Y-enemy_list[i].Y)<((70+enemy_list[i].Height)/2)-5){
            player.HP--;
            enemy_list[i].HP=0;
            if(enemy_list[i].Name=="boss") G_succeed();
            else if(enemy_list[i].Name!="boss")
                enemy_list.removeAt(i);
            if(player.HP<=0){
                t_out.stop();
                this->setCursor(Qt::ForbiddenCursor);
                GameOver=true;
                update();
                return;
            }
        }
    }//敌人与自己碰撞

    update();
    t_out.start(1);
}

void Gamewidget1::paintEvent(QPaintEvent *e){
    if(music_time==0){
        music_player=new QSoundEffect(this);
        music_player->setSource(QUrl::fromLocalFile("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/sound/bgm.wav"));
        music_player->setVolume(0.9);
        music_player->setLoopCount(QSoundEffect::Infinite);
        music_player->play();
        music_time++;
    }
    QPainter painter(this);
    if(player.level==1){
        painter.drawImage(QPoint(0,back_Y1-1080),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/background1.png"));
        back_Y1+=1.5;
    }
    else if(player.level==2){
        painter.drawImage(QPoint(0,back_Y2-1080),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/background2.png"));
        back_Y2+=1.5;
    }
    else if(player.level==3){
        painter.drawImage(QPoint(0,back_Y3-1080),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/background3.png"));
        back_Y3+=1.5;
    }
    painter.drawImage(QPoint(player.X-player.Width/2,player.Y-player.Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/playerplane.png"));


    for(int i=0;i<enemy_list.size()-1;i++){
        if(enemy_list[i].Name=="small"){
            painter.drawImage(QPoint(enemy_list[i].X-enemy_list[i].Width/2,enemy_list[i].Y-enemy_list[i].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemyplane.png"));
            enemy_list[i].X+=enemy_list[i].Speed_X;
            enemy_list[i].Y+=enemy_list[i].Speed_Y;
            if(enemy_list[i].X<=0||enemy_list[i].X>=this->width()){
                enemy_list[i].Speed_X=-enemy_list[i].Speed_X;
            }
            if(enemy_list[i].Y<0||enemy_list[i].Y>=600){
                enemy_list[i].Speed_Y=-enemy_list[i].Speed_Y;
        }
    }
        else if(enemy_list[i].Name=="mid"){
            painter.drawImage(QPoint(enemy_list[i].X-enemy_list[i].Width/2,enemy_list[i].Y-enemy_list[i].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemyplane2.png"));
            enemy_list[i].X+=enemy_list[i].Speed_X;
            enemy_list[i].Y+=enemy_list[i].Speed_Y;
            if(enemy_list[i].X<=0||enemy_list[i].X>=this->width()){
                enemy_list[i].Speed_X=-enemy_list[i].Speed_X;
            }
            if(enemy_list[i].Y<=0||enemy_list[i].Y>=this->height()){
                enemy_list[i].Speed_Y=-enemy_list[i].Speed_Y;
            }
    }
        else if(enemy_list[i].Name=="big"){
            painter.drawImage(QPoint(enemy_list[i].X-enemy_list[i].Width/2,enemy_list[i].Y-enemy_list[i].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemyplane3.png"));
            enemy_list[i].X+=enemy_list[i].Speed_X;
            if(qAbs(enemy_list[i].X-player.X)<=10){
                enemy_list[i].Speed_X=0;
            }
            if(enemy_list[i].Speed_X==0){
                enemy_list[i].Y+=enemy_list[i].Speed_Y;
            }
            if(enemy_list[i].Y>=this->height()+40)
                enemy_list.removeAt(i);
    }
        else if(enemy_list[i].Name=="boss"){
            painter.drawImage(QPoint(enemy_list[i].X-enemy_list[i].Width/2,enemy_list[i].Y-enemy_list[i].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/boss1.png"));
            enemy_list[i].X+=enemy_list[i].Speed_X;
            enemy_list[i].Y+=enemy_list[i].Speed_Y;
            if(enemy_list[i].X<=0||enemy_list[i].X>=this->width()){
                enemy_list[i].Speed_X=-enemy_list[i].Speed_X;
            }
            if(enemy_list[i].Y<=0||enemy_list[i].Y>=200){
                enemy_list[i].Speed_Y=-enemy_list[i].Speed_Y;
            }
    }
}

    for(int i=0;i<enemy_list.size()-1;i++)
        for(int j=0;j<enemy_list[i].Bullet_list.size()-1;j++){
            painter.drawImage(QPoint(enemy_list[i].Bullet_list[j].X-enemy_list[i].Bullet_list[j].Width/2,enemy_list[i].Bullet_list[j].Y+enemy_list[i].Bullet_list[j].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemybullet.png"));
            enemy_list[i].Bullet_list[j].X+=enemy_list[i].Bullet_list[j].Speed_X;
            enemy_list[i].Bullet_list[j].Y+=enemy_list[i].Bullet_list[j].Speed_Y;
            if(enemy_list[i].Bullet_list[j].Y>this->height()){
                enemy_list[i].Bullet_list.removeAt(j);
            }
        }

    for(int j=0;j<player_Bullet_list.size()-1;j++){
        painter.drawImage(QPoint(player_Bullet_list[j].X-player_Bullet_list[j].Width/2,player_Bullet_list[j].Y-player_Bullet_list[j].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/playerbullet.png"));
        player_Bullet_list[j].X+=player_Bullet_list[j].Speed_X;
        player_Bullet_list[j].Y-=player_Bullet_list[j].Speed_Y;
        if(player_Bullet_list[j].Y-player_Bullet_list[j].Height/2<0)
            player_Bullet_list.removeAt(j);
    }

    for(int i=0;i<explosion_list.size();i++){
        if(explosion_list[i].Name=="small"){
            painter.drawImage(QPoint(explosion_list[i].X-explosion_list[i].Width/2,explosion_list[i].Y-explosion_list[i].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemy1dead1.png"));
            explosion_list[i].Counter+=200;
            if(explosion_list[i].Counter>=explosion_list[i].StnTime){
                explosion_list.removeAt(i);
                count1++;
        }
        }
        else if(explosion_list[i].Name=="mid"){
            painter.drawImage(QPoint(explosion_list[i].X-explosion_list[i].Width/2,explosion_list[i].Y-explosion_list[i].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemy2dead1.png"));
            explosion_list[i].Counter+=200;
            if(explosion_list[i].Counter>=explosion_list[i].StnTime){
                explosion_list.removeAt(i);
                count1++;
          }
        }
        else if(explosion_list[i].Name=="big"){
            painter.drawImage(QPoint(explosion_list[i].X-explosion_list[i].Width/2,explosion_list[i].Y-explosion_list[i].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemy3dead1.png"));
            explosion_list[i].Counter+=200;
            if(explosion_list[i].Counter>=explosion_list[i].StnTime){
                explosion_list.removeAt(i);
                count1++;
        }
        }
    }

    if(count1>count2){
        for(int i=0;i<explosion_list2.size();i++){
            if(explosion_list2[i].Name=="small"){
                painter.drawImage(QPoint(explosion_list2[i].X-explosion_list2[i].Width/2,explosion_list2[i].Y-explosion_list2[i].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemy1dead2.png"));
                explosion_list2[i].Counter+=200;
                if(explosion_list2[i].Counter>=explosion_list2[i].StnTime){
                    explosion_list2.removeAt(i);
                    count2++;
                }

            }
            else if(explosion_list2[i].Name=="mid"){
                painter.drawImage(QPoint(explosion_list2[i].X-explosion_list2[i].Width/2,explosion_list2[i].Y-explosion_list2[i].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemy2dead2.png"));
                explosion_list2[i].Counter+=200;
                if(explosion_list2[i].Counter>=explosion_list2[i].StnTime){
                    explosion_list2.removeAt(i);
                    count2++;
                }
            }
            else if(explosion_list2[i].Name=="big"){
                painter.drawImage(QPoint(explosion_list2[i].X-explosion_list2[i].Width/2,explosion_list2[i].Y-explosion_list2[i].Height/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/enemy3dead2.png"));
                explosion_list2[i].Counter+=200;
                if(explosion_list2[i].Counter>=explosion_list2[i].StnTime){
                    explosion_list2.removeAt(i);
                    count2++;
                }
            }
        }
    }

    if(GG==1){
        painter.drawImage(QPoint(b_x-b_wid/2,b_y-b_heig/2),QImage("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/bossdead.png"));
        G_succeed();
    }

    QFont font("微软雅黑",14);
    painter.setFont(font);
    painter.setPen(Qt::red);
    painter.drawText(QPointF(10,20),"分数:"+QString::number(score));
    painter.drawText(QPointF(this->width()-90,20),"关卡:第"+QString::number(player.level)+"关");

    if(GameOver){

        setMouseTracking(true);
        setCursor(Qt::OpenHandCursor);


        label_gameover=new QLabel(this);
        label_gameover->resize(800,1080);
        label_gameover->move(0,0);
        label_gameover->setPixmap(QPixmap("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/badend.png"));

        remake=new QPushButton(label_gameover);
        exit=new QPushButton(label_gameover);

        QIcon restartbutton("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/restartbutton.png");
        QIcon quitbutton("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/quitbutton.png");


        remake->resize(200,70);
        exit->resize(200,70);
        remake->move(this->width()-500,this->height()-180);
        exit->move(this->width()-310,this->height()-180);
        remake->setIcon(restartbutton);
        exit->setIcon(quitbutton);
        remake->setIconSize(QSize(200,70));
        exit->setIconSize(QSize(200,70));
        remake->setFlat(true);
        exit->setFlat(true);
        connect(remake,SIGNAL(clicked(bool)),this,SLOT(G_remake()));
        connect(exit,SIGNAL(clicked(bool)),this,SLOT(E_exit()));

        label_gameover->show();
    }

    if(pause){
        QFont font2("微软雅黑",22);
        painter.setFont(font2);
        painter.setPen(Qt::green);
        return;
    }
}

void Gamewidget1::mouseMoveEvent(QMouseEvent *e){
    if(GameOver){
        return;
    }
    if(!pause){
        player.X=e->x();
        player.Y=e->y();
        if((player.X+player.Width/2)>this->width()) player.X=this->width()-player.Width/2;
        if((player.Y+player.Height/2)>this->height()) player.Y=this->height()-player.Height/2;
        if((player.X-player.Width/2)<=0) player.X=player.Width/2;
        if(player.Y-player.Height/2<=0) player.Y=player.Height/2;
    }
}

void Gamewidget1::keyPressEvent(QKeyEvent *e){
    if(e->key()==' '){
        pause=!pause;
        if(pause){
            t_out.stop();
            this->setCursor(Qt::ForbiddenCursor);
            update();
        }
        else {
            t_out.start(10);
            this->setCursor(Qt::BlankCursor);
            this->cursor().setPos(QPoint(player.X+this->x(),player.Y+this->y()));
        }
    }

    if(e->key()==Qt::Key_2){
        rwd_bullet_stnTime=1;
    }
    else if (e->key()==Qt::Key_1) {
        rwd_bullet_stnTime=0;

    }

    if(e->key()==Qt::Key_Escape&&key_esc==0){
        t_out.stop();
        pause=true;
        setMouseTracking(true);
        setCursor(Qt::OpenHandCursor);

        key_esc=1;


        label_pausemenu=new QLabel(this);
        label_pausemenu->resize(300,500);
        label_pausemenu->setPixmap(QPixmap("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/pausemenu.png"));
        label_pausemenu->move(this->width()/2-150,this->width()/2-250);



        QIcon buttonicon1("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/returnbutton.png");
        QIcon buttonicon2("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/quitgamebutton.png");

        restart=new QPushButton(label_pausemenu);
        exit=new QPushButton(label_pausemenu);
        restart->setFlat(true);
        exit->setFlat(true);

        restart->setMinimumSize(205,75);
        restart->setMaximumSize(205,75);
        exit->setMinimumSize(205,75);
        exit->setMaximumSize(205,75);
        restart->setIcon(buttonicon1);
        exit->setIcon(buttonicon2);
        restart->setIconSize(QSize(220,75));
        exit->setIconSize(QSize(220,75));
        restart->move(50,150);
        exit->move(50,250);



        connect(restart,SIGNAL(clicked(bool)),this,SLOT(E_restart()));
        connect(exit,SIGNAL(clicked(bool)),this,SLOT(E_exit()));

        label_pausemenu->show();
    }
}

void Gamewidget1::E_restart(){
    label_pausemenu->close();
    key_esc=0;
    setMouseTracking(false);
    setCursor(Qt::BlankCursor);
    pause=false;
    t_out.start(1);
}

void Gamewidget1::E_exit(){
    delete this;
    exit ;

}

void Gamewidget1::G_remake(){
    this->close();
    delete this;
    regame_window=new Gamewidget1();
    regame_window->show();
}

bool Gamewidget1::check_in(){
    if((player.X-player.Width/2)>=0&&(player.X+player.Height/2)<=this->width()&&(player.Y-player.Height/2)>=0&&(player.Y+player.Height/2)<=this->height())
        return true;
    else return false;
}

void Gamewidget1::G_succeed(){
    t_out.stop();
    pause=true;
    setMouseTracking(true);
    setCursor(Qt::OpenHandCursor);

    if(player.level==1){
        player.level_Up();
        score=0;

        player_Bullet_list.clear();
        for(int i=0;i<enemy_list.size()-1;i++){
            enemy_list[i].Bullet_list.clear();
        }
        enemy_list.clear();
        explosion_list.clear();
        explosion_list2.clear();

        label_next=new QLabel(this);
        label_next->resize(this->width(),this->height());
        label_next->move(0,0);
        label_next->setPixmap(QPixmap("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/diary2.png"));

        QIcon nextbutton("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/nextbutton.png");

        next=new QPushButton(label_next);
        next->resize(200,75);
        next->setIcon(nextbutton);
        next->setIconSize(QSize(200,75));
        next->setFlat(true);
        next->move(this->width()-380,this->height()-180);

        label_next->show();

        connect(next,SIGNAL(clicked(bool)),this,SLOT(G_next()));
        connect(next,SIGNAL(clicked(bool)),label_next,SLOT(close()));

    }
    else if(player.level==2){
        player.level_Up();

        score=0;

        player_Bullet_list.clear();
        for(int i=0;i<enemy_list.size()-1;i++){
            enemy_list[i].Bullet_list.clear();
        }
        enemy_list.clear();
        explosion_list.clear();
        explosion_list2.clear();

        label_next2=new QLabel(this);
        label_next2->resize(this->width(),this->height());
        label_next2->move(0,0);
        label_next2->setPixmap(QPixmap("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/diary3.png"));

        QIcon nextbutton("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/nextbutton.png");

        next=new QPushButton(label_next2);
        next->resize(200,75);
        next->setIcon(nextbutton);
        next->setIconSize(QSize(200,75));
        next->setFlat(true);
        next->move(this->width()-380,this->height()-180);

        label_next2->show();

        connect(next,SIGNAL(clicked(bool)),this,SLOT(G_next()));
        connect(next,SIGNAL(clicked(bool)),label_next2,SLOT(close()));

    }
        else if(player.level==3){
        player_Bullet_list.clear();
        for(int i=0;i<enemy_list.size()-1;i++){
            enemy_list[i].Bullet_list.clear();
        }
        enemy_list.clear();
        explosion_list.clear();
        explosion_list2.clear();
        if(player.HP<=0){
            label_egg=new QLabel(this);
            label_egg->resize(this->width(),this->height());
            label_egg->move(0,0);
            label_egg->setPixmap(QPixmap("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/end2.png"));

            remake=new QPushButton(label_egg);
            exit=new QPushButton(label_egg);

            QIcon restartbutton("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/restartbutton.png");
            QIcon quitbutton("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/quitbutton.png");


            remake->resize(200,70);
            exit->resize(200,70);
            remake->move(this->width()-500,this->height()-180);
            exit->move(this->width()-310,this->height()-180);
            remake->setIcon(restartbutton);
            exit->setIcon(quitbutton);
            remake->setIconSize(QSize(200,70));
            exit->setIconSize(QSize(200,70));
            remake->setFlat(true);
            exit->setFlat(true);

            label_egg->show();

            connect(remake,SIGNAL(clicked(bool)),this,SLOT(G_remake()));
            connect(exit,SIGNAL(clicked(bool)),this,SLOT(E_exit()));
        }//达成结局：?
        else {
            label_succeed=new QLabel(this);
            label_succeed->resize(this->width(),this->height());
            label_succeed->move(0,0);
            label_succeed->setPixmap(QPixmap("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/end1.png"));

            remake=new QPushButton(label_succeed);
            exit=new QPushButton(label_succeed);

            QIcon restartbutton("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/restartbutton.png");
            QIcon quitbutton("E:/qt/build-MyWar-Desktop_Qt_5_10_1_MinGW_32bit-Debug/images/quitbutton.png");


            remake->resize(200,70);
            exit->resize(200,70);
            remake->move(this->width()-500,this->height()-180);
            exit->move(this->width()-310,this->height()-180);
            remake->setIcon(restartbutton);
            exit->setIcon(quitbutton);
            remake->setIconSize(QSize(200,70));
            exit->setIconSize(QSize(200,70));
            remake->setFlat(true);
            exit->setFlat(true);

            label_succeed->show();

            connect(remake,SIGNAL(clicked(bool)),this,SLOT(G_remake()));
            connect(exit,SIGNAL(clicked(bool)),this,SLOT(E_exit()));
        }//达成结局：大胜而归

    }
}

void Gamewidget1::G_next(){
    pause=false;
    t_out.start(1);
    setMouseTracking(false);
    this->setCursor(Qt::BlankCursor);
}
