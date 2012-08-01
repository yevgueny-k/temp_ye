#include <QSharedPointer>
#include <QTextStream>
#include <QRectF>
#include <QPainter>
#include <qmath.h>

#include <QEMath.h>
#include <QEIDesignSnapController.h>

#include "RadialSnap.h"

// Utility functions (necessary for this particular type of snap adapter, see snap()).
namespace _local_RadialSnap
{
  // For prec>0:
  //   Rounds value < 10^prec to have number of significant digits == prec.
  //   Value >= 10^prec is rounded to integer.
  // For prec<=0:
  //   Rounds any value to integer setting -prec lesser digits of this integer to zero.
  inline double smartRoundInt(double value, int prec=3){
      if(std::fabs(value)<1.e-20)return 0;
      double mult(prec>0
          ?(std::fabs(value)>=std::pow(10., prec)
              ?1.
              :std::pow(10., std::floor(std::log10(std::fabs(value)))-prec+1))
          :std::pow(10., -prec));
      return std::floor(value/mult+0.5)*mult;
  }

  inline qreal qDistance(qreal x1, qreal y1, qreal x2, qreal y2)
  {
    return qSqrt(qPow(x2 - x1, 2) + qPow(y2 - y1, 2));
  }

  template<class T>
  inline qreal qDistance(const T& p1, const T& p2)
  {
    return qDistance(p1.x(), p1.y(), p2.x(), p2.y());
  }

  QPointF projectedPoint(const QPointF& sourcePoint, const QLineF& targetLine)
  {
    QLineF perpendicular(sourcePoint, sourcePoint + QPointF(targetLine.dy(), -targetLine.dx()));
    QPointF intersection;
    if (perpendicular.intersect(targetLine, &intersection) == QLineF::NoIntersection) return sourcePoint;
    return intersection;
  }

  double perpendicularLength(const QPointF& sourcePoint, const QLineF& targetLine)
  {
    return qDistance(sourcePoint, projectedPoint(sourcePoint,targetLine));
  }

}
using namespace _local_RadialSnap;

RadialSnapAdapter::RadialSnapAdapter(QEIDesignDocument* d)
:
  mDocument(d)
{
}

// -- QEIInterface
QString RadialSnapAdapter::uid() const
{
  return "com.quasado.addins.debug.RadialSnapAdapter";
}

void RadialSnapAdapter::restoreSettings(const QSettings& settings, unsigned int context)
{
  // not implemented
}

void RadialSnapAdapter::saveSettings(QSettings& settings, unsigned int context) const
{
  // not implemented
}

QSharedPointer<QEIDesignSnapPara> RadialSnapAdapter::snapDefaultPara() const
{
  // trivial

  return QSharedPointer<QEIDesignSnapPara>(new RadialSnapPara(mDocument, true));
}

void RadialSnapAdapter::snap(QSharedPointer<QEIDesignSnapDescriptor> context)
{
  // the most important function in both RadialSnap* classes

  if (!mDocument) return;

  // Check if the given context is correct.
  if (context.isNull()) return;
  if (
    !(
      context->info()->isOn()
      && context->info()->snapType() == kUserDesignSnapAdapter
      && context->info()->snapTypeName() == "Radial Snap"
    )
  )
  {
    return;
  }

  // Clear the results in the given context.
  context->clearResults();

  // Access the snap-specific prameters.
  RadialSnapPara* paraImpl = static_cast<RadialSnapPara*>(context->info().data());

  // The snap algorithm.
  double angle = QLineF(paraImpl->paramRaySourcePoint(), context->sourcePoint()).angle();
  double rayAngle = int (angle / paraImpl->paramAngleStep() + 0.5) * paraImpl->paramAngleStep();
  QLineF ray = QLineF::fromPolar(1., rayAngle);
  QPointF projection = projectedPoint(context->sourcePoint(), ray);

  if (
      qeMaxAxisDistance(context->sourcePoint(), projection)
      <= mDocument->snapController()->varMaxSnapDistance(mDocument->snapController()->zoomOfActiveView())
  )
  {
    context->setOffset(projection - context->sourcePoint());

// Needed only if the point that was snapped is different from context->sourcePoint() (i. e. it's shape anchor point etc.)
//    context->setSnapOrigin(QEIDesignSnapDescriptor::kSnapOriginPoint);
//    context->setSnapOriginPoint(...);

    QString s;
      QTextStream ts(&s);
      ts << "ray (" << rayAngle << " deg. from " << smartRoundInt(paraImpl->paramRaySourcePoint().x(), 1) << ":" << smartRoundInt(paraImpl->paramRaySourcePoint().x(), 1) << ")";
    context->setHintText(s);

    context->setIsSnapped(true);

// =======================

// The follwing three properties are optional, but if set,
//  allow combinatoric controller to intersect target lines from different kinds of snaps.
// This option is useful to snaps which are, geometrically, line segments.
    context->setLineHintText(s);
    context->setLineSegment(ray.translated(paraImpl->paramRaySourcePoint()));

    context->setLineIsSnapped(true);

// =======================

  }

}

QList<QEUpdatingRect> RadialSnapAdapter::snapGetPaintAreas(QSharedPointer<QEIDesignSnapDescriptor> context, const QList<QEIView*>& destViews)
{
  QList<QEUpdatingRect> retval;

  if (!mDocument) return retval;

  // Check if the given context is correct.
  if (context.isNull()) return retval;
  if (
    !(
      context->info()->isOn()
      && context->info()->snapType() == kUserDesignSnapAdapter
      && context->info()->snapTypeName() == "Radial Snap"
    )
  )
  {
    return retval;
  }

  // snap-specific

  // Access the snap-specific prameters.
  RadialSnapPara* paraImpl = static_cast<RadialSnapPara*>(context->info().data());

  retval.append(QEUpdatingRect(QRectF(paraImpl->paramRaySourcePoint(), context->snappedPoint()).adjusted(-1., -1., 1., 1.), 0));

  // generic visual point-kind snap representation

  retval.append(mDocument->snapController()->snapGetPaintAreas(context, destViews));

  return retval;
}

void RadialSnapAdapter::snapPaint(QSharedPointer<QEIDesignSnapDescriptor> context, QPainter* painter, const QRectF& rect)
{
  if (!mDocument) return;

  // Check if the given context is correct.
  if (context.isNull()) return;
  if (
    !(
      context->info()->isOn()
      && context->info()->snapType() == kUserDesignSnapAdapter
      && context->info()->snapTypeName() == "Radial Snap"
    )
  )
  {
    return;
  }

  QEIDesignSnapController* c = mDocument->snapController();
  QRectF scale = painter->combinedTransform().mapRect(QRectF(0., 0., 1., 1.));

  // snap-specific

  RadialSnapPara* paraImpl = static_cast<RadialSnapPara*>(context->info().data());
  if (rect.intersects(QRectF(paraImpl->paramRaySourcePoint(), context->snappedPoint()).adjusted(-1., -1., 1., 1.)))
  {
    painter->setPen(c->varPenTarget(scale.width()));
    painter->drawLine(paraImpl->paramRaySourcePoint(), context->snappedPoint());
  }

  // generic visual point-kind snap representation

  c->snapPaint(context, painter, rect);
}

QString RadialSnapPara::uid() const
{
  return "com.quasado.addins.debug.RadialSnapPara";
}

void RadialSnapPara::restoreSettings(const QSettings& settings, unsigned int context)
{
  // not implemented
}

void RadialSnapPara::saveSettings(QSettings& settings, unsigned int context) const
{
  // not implemented
}

void RadialSnapPara::setParamAngleStep(double x)
{
  // snap-specific

  if (x < 0.001)
  {
    x = 0.001;
  }
  mParamAngleStep = x;
}

void RadialSnapPara::setParamRaySourcePoint(const QPointF& x)
{
  // snap-specific

  mParamRaySourcePoint = x;
}

RadialSnapPara::RadialSnapPara(QEIDesignDocument* d, bool isOn)
:
  mIsOn(isOn),
  mDocument(d),
  mParamAngleStep(20.),
  mParamRaySourcePoint(0., 0.)
{
}

QSharedPointer<QEIDesignSnapPara> RadialSnapPara::clone() const
{
  // trivial copying

  RadialSnapPara* retval = new RadialSnapPara(mDocument, mIsOn);

  retval->mParamAngleStep = mParamAngleStep;
  retval->mParamRaySourcePoint = mParamRaySourcePoint;

  return QSharedPointer<QEIDesignSnapPara>(retval);
}
